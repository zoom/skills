# Angular Implementation Guide

Complete Angular 21 + Vite 7 + TypeScript + Standalone Components implementation for Zoom Video SDK.

## Prerequisites

- **Bun** (recommended) or npm/yarn
- **Node.js** 24+
- **Angular CLI** 21+
- Basic knowledge of Angular standalone components and signals

## Project Setup

### 1. Create Angular Project

```bash
# Install Angular CLI globally
npm install -g @angular/cli

# Create new Angular project with standalone components
ng new my-zoom-app-angular --standalone --routing --style=css
cd my-zoom-app-angular
```

### 2. Install Dependencies

```bash
# Core dependencies
npm install @zoom/videosdk

# Icon library (recommended)
npm install lucide-angular

# UI library (Angular Material or PrimeNG recommended)
ng add @angular/material

# Or use PrimeNG
npm install primeng primeicons
npm install @primeng/themes

# Development tools
npm install -D @biomejs/biome
npx biome init
```

### 3. Configure Tailwind CSS (Optional)

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init
```

```css
/* src/styles.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## CRITICAL: Vite Configuration (if using Vite)

If you're using Angular with Vite (Angular 17+), configure COOP/COEP headers:

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import angular from "@analogjs/vite-plugin-angular";

export default defineConfig({
  plugins: [angular()],
  server: {
    headers: {
      // required for SharedArrayBuffer for gallery view if not use video_webrtc_mode
      "Cross-Origin-Opener-Policy": "same-origin",
      "Cross-Origin-Embedder-Policy": "require-corp",
    },
  },
});
```

## CRITICAL: Custom Elements Schema

```typescript
// src/app/app.config.ts
import { ApplicationConfig, CUSTOM_ELEMENTS_SCHEMA } from "@angular/core";
import { provideRouter } from "@angular/router";
import { provideLucideIcons } from "lucide-angular";
import {
  Headphones,
  Info,
  LogOut,
  MessageSquare,
  Mic,
  MicOff,
  Monitor,
  MonitorOff,
  Users,
  Video,
  VideoOff,
} from "lucide-angular/icons";
import { routes } from "./app.routes";

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    // Provide lucide icons
    provideLucideIcons({
      Headphones,
      Info,
      LogOut,
      MessageSquare,
      Mic,
      MicOff,
      Monitor,
      MonitorOff,
      Users,
      Video,
      VideoOff,
    }),
  ],
  // Add this to support Zoom's custom elements
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
};
```

Or for component-level:

```typescript
// In each component that uses video-player-container
@Component({
  // ...
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
})
```

## CRITICAL: Angular Zone Configuration

**You MUST configure Angular zones** to avoid performance issues with Zoom Video SDK. The SDK must run outside the Angular zone to prevent unnecessary change detection.

https://developers.zoom.us/docs/video-sdk/web/frameworks/#using-angular

### 1. Create zone-flags.ts

In the `src` directory, create a `zone-flags.ts` file:

```typescript
// src/zone-flags.ts
// Disable patching requestAnimationFrame
(window as any).__Zone_disable_requestAnimationFrame = true;

// Disable patching specified eventNames
(window as any).__zone_symbol__UNPATCHED_EVENTS = ["message"];
```

### 2. Update angular.json

Add `zone-flags.ts` to the polyfills array **before** `zone.js`:

```json
{
  "projects": {
    "your-app": {
      "architect": {
        "build": {
          "options": {
            "polyfills": ["src/zone-flags.ts", "zone.js"]
          }
        }
      }
    }
  }
}
```

### 3. Update tsconfig.app.json

Add `zone-flags.ts` to the include array:

```json
{
  "include": ["src/**/*.d.ts", "src/zone-flags.ts"]
}
```

### 4. Run SDK Outside Angular Zone

**CRITICAL**: All Zoom SDK operations must run outside the Angular zone using `NgZone.runOutsideAngular()`:

```typescript
import { Injectable, NgZone } from "@angular/core";
import ZoomVideo, { type VideoClient } from "@zoom/videosdk";

@Injectable({
  providedIn: "root",
})
export class ZoomClientService {
  private clientInstance: typeof VideoClient | null = null;

  constructor(private ngZone: NgZone) {}

  async init(): Promise<typeof VideoClient> {
    // CRITICAL: Run outside Angular zone
    return this.ngZone.runOutsideAngular(async () => {
      const zoomClient = this.getClient();
      await zoomClient.init("en-US", "Global", {
        patchJsMedia: true,
        leaveOnPageUnload: true,
      });
      return zoomClient;
    });
  }

  async join(
    sessionName: string,
    token: string,
    userName: string,
    password?: string
  ): Promise<typeof VideoClient> {
    // CRITICAL: Run outside Angular zone
    return this.ngZone.runOutsideAngular(async () => {
      const zoomClient = this.getClient();
      await zoomClient.join(sessionName, token, userName, password);
      return zoomClient;
    });
  }
}
```

**Why this is required:**

- Angular's zone.js patches browser APIs like `requestAnimationFrame` and event listeners
- Zoom Video SDK uses these APIs extensively for video rendering
- Running inside Angular zone causes excessive change detection cycles
- This leads to severe performance degradation and UI freezing
- Running outside the zone prevents Angular from tracking these operations

## Project Structure

```
src/
├── app/
│   ├── components/
│   │   ├── video-session/
│   │   │   ├── session-status.component.ts
│   │   │   ├── video-grid.component.ts
│   │   │   ├── participant-list.component.ts
│   │   │   ├── chat-panel.component.ts
│   │   │   ├── control-bar.component.ts
│   │   │   ├── session-info.component.ts
│   │   │   ├── join-form.component.ts
│   │   │   └── session-page.component.ts
│   ├── services/
│   │   ├── zoom-client.service.ts
│   │   ├── media-stream.service.ts
│   │   ├── participants.service.ts
│   │   └── chat.service.ts
│   ├── models/
│   │   └── zoom.types.ts
│   └── app.component.ts
└── styles.css
```

## Types

```typescript
// src/app/models/zoom.types.ts
export type ViewMode = "speaker" | "gallery";
export type ConnectionStatus =
  | "Disconnected"
  | "Connecting"
  | "Connected"
  | "Reconnecting"
  | "Failed";

export interface ChatMessage {
  id: string;
  sender: { name: string; userId: number };
  message: string;
  timestamp: number;
  isPrivate: boolean;
}
```

## Services

### zoom-client.service.ts

```typescript
import { Injectable, signal, NgZone } from "@angular/core";
import ZoomVideo, { type VideoClient } from "@zoom/videosdk";

@Injectable({
  providedIn: "root",
})
export class ZoomClientService {
  private clientInstance: typeof VideoClient | null = null;

  client = signal<typeof VideoClient | null>(null);
  isInitialized = signal(false);
  isJoined = signal(false);

  constructor(private ngZone: NgZone) {}

  getClient(): typeof VideoClient {
    if (!this.clientInstance) {
      this.clientInstance = ZoomVideo.createClient();
      this.client.set(this.clientInstance);
    }
    return this.clientInstance;
  }

  async init(): Promise<typeof VideoClient> {
    // CRITICAL: Run outside Angular zone to prevent performance issues
    return this.ngZone.runOutsideAngular(async () => {
      const zoomClient = this.getClient();
      await zoomClient.init("en-US", "Global", {
        patchJsMedia: true,
        leaveOnPageUnload: true,
      });

      // Update signals inside Angular zone
      this.ngZone.run(() => {
        this.isInitialized.set(true);
      });

      zoomClient.on("connection-change", (payload) => {
        if (payload.state === "Closed" || payload.state === "Fail") {
          // Update signals inside Angular zone
          this.ngZone.run(() => {
            this.isJoined.set(false);
          });
        }
      });

      return zoomClient;
    });
  }

  async join(
    sessionName: string,
    token: string,
    userName: string,
    password?: string
  ): Promise<typeof VideoClient> {
    // CRITICAL: Run outside Angular zone
    return this.ngZone.runOutsideAngular(async () => {
      const zoomClient = this.getClient();
      if (!this.isInitialized()) {
        await this.init();
      }
      await zoomClient.join(sessionName, token, userName, password);

      // Update signals inside Angular zone
      this.ngZone.run(() => {
        this.isJoined.set(true);
        this.client.set(zoomClient);
      });

      return zoomClient;
    });
  }

  async leave(endSession = false): Promise<void> {
    // CRITICAL: Run outside Angular zone
    return this.ngZone.runOutsideAngular(async () => {
      if (this.clientInstance) {
        await this.clientInstance.leave(endSession);

        // Update signals inside Angular zone
        this.ngZone.run(() => {
          this.isJoined.set(false);
        });
      }
    });
  }

  destroy(): void {
    if (this.clientInstance) {
      ZoomVideo.destroyClient();
      this.clientInstance = null;
      this.client.set(null);
      this.isInitialized.set(false);
      this.isJoined.set(false);
    }
  }
}
```

### media-stream.service.ts

```typescript
import { Injectable, signal, effect } from "@angular/core";
import type { Stream } from "@zoom/videosdk";
import { ZoomClientService } from "./zoom-client.service";

@Injectable({
  providedIn: "root",
})
export class MediaStreamService {
  stream = signal<typeof Stream | null>(null);
  audioOn = signal(false);
  videoOn = signal(false);
  sharing = signal(false);
  muted = signal(true);

  constructor(private zoomClient: ZoomClientService) {
    // Listen for client changes and attach event listeners
    effect(() => {
      const client = this.zoomClient.client();
      if (!client) return;

      const audioHandler = (payload: { action: string }) => {
        if (payload.action === "join") {
          this.audioOn.set(true);
          this.muted.set(true);
        } else if (payload.action === "leave") {
          this.audioOn.set(false);
        } else if (payload.action === "muted") {
          this.muted.set(true);
        } else if (payload.action === "unmuted") {
          this.muted.set(false);
        }
      };

      // CRITICAL: Sync video state with SDK events
      const videoActiveHandler = (payload: {
        state: string;
        userId: number;
      }) => {
        const currentUser = client.getCurrentUserInfo();
        if (currentUser && payload.userId === currentUser.userId) {
          this.videoOn.set(payload.state === "Active");
        }
      };

      const shareHandler = (payload: { state: string; userId: number }) => {
        const currentUser = client.getCurrentUserInfo();
        if (currentUser && payload.userId === currentUser.userId) {
          this.sharing.set(payload.state === "Active");
        }
      };

      client.on("current-audio-change", audioHandler);
      client.on("video-active-change", videoActiveHandler);
      client.on("active-share-change", shareHandler);
    });
  }

  initStream(): typeof Stream | null {
    const client = this.zoomClient.client();
    if (client) {
      const mediaStream = client.getMediaStream();
      this.stream.set(mediaStream);
      return mediaStream;
    }
    return null;
  }

  async toggleAudio(): Promise<void> {
    const currentStream = this.stream();
    if (!currentStream) throw new Error("Stream not initialized");

    if (!this.audioOn()) {
      await currentStream.startAudio();
    } else if (this.muted()) {
      await currentStream.unmuteAudio();
    } else {
      await currentStream.muteAudio();
    }
  }

  async toggleVideo(): Promise<void> {
    const currentStream = this.stream();
    if (!currentStream) throw new Error("Stream not initialized");

    if (this.videoOn()) {
      await currentStream.stopVideo();
      this.videoOn.set(false);
    } else {
      await currentStream.startVideo();
      this.videoOn.set(true);
    }
  }

  async toggleScreenShare(): Promise<void> {
    const currentStream = this.stream();
    if (!currentStream) throw new Error("Stream not initialized");

    if (this.sharing()) {
      await currentStream.stopShareScreen();
      this.sharing.set(false);
    } else {
      await currentStream.startShareScreen();
      this.sharing.set(true);
    }
  }
}
```

### participants.service.ts

```typescript
import { Injectable, signal, effect } from "@angular/core";
import type { Participant } from "@zoom/videosdk";
import { ZoomClientService } from "./zoom-client.service";

@Injectable({
  providedIn: "root",
})
export class ParticipantsService {
  participants = signal<Participant[]>([]);
  activeSpeakerId = signal<number | null>(null);

  constructor(private zoomClient: ZoomClientService) {
    // Listen for client changes and attach event listeners
    effect(() => {
      const client = this.zoomClient.client();
      if (!client) return;

      const updateList = () => {
        this.participants.set(client.getAllUser());
      };

      client.on("user-added", updateList);
      client.on("user-updated", updateList);
      client.on("user-removed", updateList);

      // CRITICAL: Listen for video state changes to refresh participant list
      client.on("video-active-change", updateList);
      client.on("peer-video-state-change", updateList);

      const connectionHandler = (payload: { state: string }) => {
        if (payload.state === "Connected") updateList();
      };
      client.on("connection-change", connectionHandler);

      // Active speaker tracking
      const speakerHandler = (payload: {
        activeSpeaker: Array<{ oderId: number }>;
      }) => {
        if (payload.activeSpeaker.length > 0) {
          this.activeSpeakerId.set(payload.activeSpeaker[0].oderId);
        }
      };
      client.on("active-speaker", speakerHandler);
    });

    // Update participants when joined
    effect(() => {
      if (this.zoomClient.isJoined() && this.zoomClient.client()) {
        setTimeout(() => {
          const client = this.zoomClient.client();
          if (client) {
            this.participants.set(client.getAllUser());
          }
        }, 100);
      }
    });
  }
}
```

### chat.service.ts

```typescript
import { Injectable, signal, effect } from "@angular/core";
import { ZoomClientService } from "./zoom-client.service";
import type { ChatMessage } from "../models/zoom.types";

@Injectable({
  providedIn: "root",
})
export class ChatService {
  messages = signal<ChatMessage[]>([]);

  constructor(private zoomClient: ZoomClientService) {
    effect(() => {
      const client = this.zoomClient.client();
      if (!client) return;

      const handler = (payload: ChatMessage) => {
        this.messages.update((msgs) => [
          ...msgs,
          { ...payload, id: crypto.randomUUID() },
        ]);
      };
      client.on("chat-on-message", handler);
    });
  }

  async sendMessage(text: string): Promise<void> {
    const client = this.zoomClient.client();
    if (!client) return;
    const chatClient = client.getChatClient();
    await chatClient.send(text);
  }

  clearMessages(): void {
    this.messages.set([]);
  }
}
```

## Components

### session-status.component.ts

```typescript
import { Component, input } from "@angular/core";
import { CommonModule } from "@angular/common";
import type { ConnectionStatus } from "../../models/zoom.types";

@Component({
  selector: "app-session-status",
  standalone: true,
  imports: [CommonModule],
  template: ` <span [class]="badgeClass()">{{ status() }}</span> `,
  styles: [
    `
      span {
        padding: 0.25rem 0.75rem;
        border-radius: 0.375rem;
        font-size: 0.875rem;
        font-weight: 500;
      }
      .connected {
        background-color: #22c55e;
        color: white;
      }
      .connecting {
        background-color: #eab308;
        animation: pulse 2s infinite;
      }
      .reconnecting {
        background-color: #eab308;
        animation: pulse 2s infinite;
      }
      .disconnected {
        background-color: #6b7280;
        color: white;
      }
      .failed {
        background-color: #ef4444;
        color: white;
      }
      @keyframes pulse {
        0%,
        100% {
          opacity: 1;
        }
        50% {
          opacity: 0.5;
        }
      }
    `,
  ],
})
export class SessionStatusComponent {
  status = input.required<ConnectionStatus>();

  badgeClass() {
    return this.status().toLowerCase();
  }
}
```

### video-grid.component.ts

```typescript
import {
  Component,
  input,
  effect,
  ElementRef,
  ViewChild,
  CUSTOM_ELEMENTS_SCHEMA,
} from "@angular/core";
import { CommonModule } from "@angular/common";
import type { Participant, Stream } from "@zoom/videosdk";
import type { ViewMode } from "../../models/zoom.types";

@Component({
  selector: "app-video-tile",
  standalone: true,
  imports: [CommonModule],
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
  template: `
    <div [class]="containerClass()">
      <video-player-container
        #videoContainer
        [style.width]="'100%'"
        [style.height]="'100%'"
        [style.display]="isVideoOn() ? 'block' : 'none'"
      ></video-player-container>

      @if (!isVideoOn()) {
      <div class="avatar-container">
        <div [class]="avatarClass()">
          {{ participant().displayName.charAt(0).toUpperCase() }}
        </div>
        <svg
          class="video-off-icon"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
        >
          <path
            d="M16 16v1a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V7a2 2 0 0 1 2-2h2m5.66 0H14a2 2 0 0 1 2 2v3.34l1 1L23 7v10"
          />
          <line x1="1" y1="1" x2="23" y2="23" />
        </svg>
      </div>
      }

      <div class="participant-badge">
        {{ participant().displayName
        }}{{ participant().isHost ? " (Host)" : "" }}
      </div>
    </div>
  `,
  styles: [
    `
      :host {
        display: block;
        position: relative;
        background-color: #1f2937;
        border-radius: 0.5rem;
        overflow: hidden;
      }
      .container {
        position: relative;
        display: flex;
        align-items: center;
        justify-content: center;
        width: 100%;
        height: 100%;
      }
      .container.large {
        width: 100%;
        height: 100%;
      }
      .container.small {
        aspect-ratio: 16/9;
      }
      .container.speaking {
        box-shadow: 0 0 0 2px #22c55e;
      }
      .avatar-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 0.5rem;
      }
      .avatar {
        display: flex;
        align-items: center;
        justify-content: center;
        border-radius: 50%;
        background-color: #4b5563;
        color: white;
        font-weight: 600;
      }
      .avatar.large {
        width: 6rem;
        height: 6rem;
        font-size: 1.875rem;
      }
      .avatar.small {
        width: 4rem;
        height: 4rem;
        font-size: 1.25rem;
      }
      .video-off-icon {
        width: 1.25rem;
        height: 1.25rem;
        color: #9ca3af;
      }
      .participant-badge {
        position: absolute;
        bottom: 0.5rem;
        left: 0.5rem;
        background-color: rgba(0, 0, 0, 0.7);
        color: white;
        padding: 0.25rem 0.5rem;
        border-radius: 0.25rem;
        font-size: 0.75rem;
        max-width: 80%;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }
    `,
  ],
})
export class VideoTileComponent {
  participant = input.required<Participant>();
  stream = input<typeof Stream | null>(null);
  isLarge = input(false);
  isSpeaking = input(false);
  currentUserId = input<number>();
  currentUserVideoOn = input(false);

  @ViewChild("videoContainer", { read: ElementRef })
  videoContainer?: ElementRef;

  isVideoOn() {
    const isCurrentUser = this.currentUserId() === this.participant().userId;
    return isCurrentUser
      ? this.currentUserVideoOn()
      : this.participant().bVideoOn;
  }

  containerClass() {
    return `container ${this.isLarge() ? "large" : "small"} ${
      this.isSpeaking() ? "speaking" : ""
    }`;
  }

  avatarClass() {
    return `avatar ${this.isLarge() ? "large" : "small"}`;
  }

  constructor() {
    effect(() => {
      this.attachVideo();
    });
  }

  async attachVideo() {
    const streamVal = this.stream();
    const container = this.videoContainer?.nativeElement;
    if (!streamVal || !container) return;

    if (!this.isVideoOn()) {
      try {
        streamVal.detachVideo(this.participant().userId);
        container.innerHTML = "";
      } catch {
        // Ignore detach errors
      }
      return;
    }

    try {
      const videoElement = await streamVal.attachVideo(
        this.participant().userId,
        3 // VideoQuality.Video_720P
      );

      if (videoElement) {
        container.innerHTML = "";
        const el = videoElement as unknown as HTMLElement;
        el.style.width = "100%";
        el.style.height = "100%";
        container.appendChild(el);
      }
    } catch (error) {
      console.error("Error attaching video:", error);
    }
  }

  ngOnDestroy() {
    const streamVal = this.stream();
    if (streamVal) {
      try {
        streamVal.detachVideo(this.participant().userId);
      } catch {
        // Ignore detach errors
      }
    }
  }
}

@Component({
  selector: "app-video-grid",
  standalone: true,
  imports: [CommonModule, VideoTileComponent],
  template: `
    @if (participants().length === 0) {
    <div class="empty-state">
      <p>Waiting for participants...</p>
    </div>
    } @else if (viewMode() === 'speaker') {
    <div class="speaker-view">
      <app-video-tile
        [participant]="speaker()"
        [stream]="stream()"
        [isLarge]="true"
        [isSpeaking]="speaker().userId === activeSpeakerId()"
        [currentUserId]="currentUserId()"
        [currentUserVideoOn]="currentUserVideoOn()"
      />
      @if (others().length > 0) {
      <div class="thumbnails">
        @for (p of others().slice(0, 4); track p.userId) {
        <div class="thumbnail">
          <app-video-tile
            [participant]="p"
            [stream]="stream()"
            [isSpeaking]="p.userId === activeSpeakerId()"
            [currentUserId]="currentUserId()"
            [currentUserVideoOn]="currentUserVideoOn()"
          />
        </div>
        }
      </div>
      }
    </div>
    } @else {
    <div [class]="'gallery-view ' + getGridCols()">
      @for (p of participants(); track p.userId) {
      <app-video-tile
        [participant]="p"
        [stream]="stream()"
        [isSpeaking]="p.userId === activeSpeakerId()"
        [currentUserId]="currentUserId()"
        [currentUserVideoOn]="currentUserVideoOn()"
      />
      }
    </div>
    }
  `,
  styles: [
    `
      :host {
        display: block;
        height: 100%;
      }
      .empty-state {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 100%;
        color: #9ca3af;
      }
      .speaker-view {
        position: relative;
        height: 100%;
        padding: 1rem;
      }
      .thumbnails {
        position: absolute;
        bottom: 1.5rem;
        left: 1.5rem;
        display: flex;
        gap: 0.5rem;
      }
      .thumbnail {
        width: 8rem;
      }
      .gallery-view {
        display: grid;
        gap: 0.5rem;
        height: 100%;
        padding: 1rem;
      }
      .grid-cols-1 {
        grid-template-columns: repeat(1, 1fr);
      }
      .grid-cols-2 {
        grid-template-columns: repeat(2, 1fr);
      }
      .grid-cols-3 {
        grid-template-columns: repeat(3, 1fr);
      }
      .grid-cols-4 {
        grid-template-columns: repeat(4, 1fr);
      }
    `,
  ],
})
export class VideoGridComponent {
  participants = input.required<Participant[]>();
  viewMode = input.required<ViewMode>();
  activeSpeakerId = input<number | null>(null);
  stream = input<typeof Stream | null>(null);
  currentUserId = input<number>();
  currentUserVideoOn = input(false);

  speaker() {
    return (
      this.participants().find((p) => p.userId === this.activeSpeakerId()) ||
      this.participants()[0]
    );
  }

  others() {
    return this.participants().filter(
      (p) => p.userId !== this.speaker().userId
    );
  }

  getGridCols() {
    const count = this.participants().length;
    if (count === 1) return "grid-cols-1";
    if (count <= 4) return "grid-cols-2";
    if (count <= 9) return "grid-cols-3";
    return "grid-cols-4";
  }
}
```

### control-bar.component.ts

```typescript
import { Component, input, output } from "@angular/core";
import { CommonModule } from "@angular/common";
import { LucideAngularModule } from "lucide-angular";

@Component({
  selector: "app-control-bar",
  standalone: true,
  imports: [CommonModule, LucideAngularModule],
  template: `
    <div class="control-bar">
      <button
        [class]="audioOn() && !muted() ? 'active' : ''"
        (click)="toggleAudio.emit()"
        [title]="!audioOn() || muted() ? 'Unmute' : 'Mute'"
      >
        @if (!audioOn()) {
          <i-lucide name="headphones" />
        } @else if (muted()) {
          <i-lucide name="mic-off" />
        } @else {
          <i-lucide name="mic" />
        }
      </button>
      <button
        [class]="videoOn() ? 'active' : ''"
        (click)="toggleVideo.emit()"
        [title]="videoOn() ? 'Stop Video' : 'Start Video'"
      >
        @if (videoOn()) {
          <i-lucide name="video" />
        } @else {
          <i-lucide name="video-off" />
        }
      </button>
      <button
        [class]="sharing() ? 'active' : ''"
        (click)="toggleShare.emit()"
        [title]="sharing() ? 'Stop Share' : 'Share Screen'"
      >
        @if (sharing()) {
          <i-lucide name="monitor-off" />
        } @else {
          <i-lucide name="monitor" />
        }
      </button>
      <div class="divider"></div>
      <button
        [class]="showParticipants() ? 'active' : ''"
        (click)="toggleParticipants.emit()"
        title="Participants"
      >
        <i-lucide name="users" />
      </button>
      <button
        [class]="showChat() ? 'active' : ''"
        (click)="toggleChat.emit()"
        title="Chat"
      >
        <i-lucide name="message-square" />
      </button>
      <button (click)="toggleInfo.emit()" title="Info">
        <i-lucide name="info" />
      </button>
      <div class="divider"></div>
      <button class="leave" (click)="leave.emit()">
        <i-lucide name="log-out" />
        Leave
      </button>
    </div>
  `,
  styles: [
    `
      .control-bar {
        display: flex;
        align-items: center;
        justify-content: center;
        gap: 0.5rem;
        padding: 1rem;
        border-top: 1px solid #e5e7eb;
        background-color: white;
      }
      button {
        padding: 0.5rem 1rem;
        border: none;
        border-radius: 0.375rem;
        background-color: #f3f4f6;
        cursor: pointer;
        font-size: 1.25rem;
      }
      button:hover {
        background-color: #e5e7eb;
      }
      button.active {
        background-color: #3b82f6;
        color: white;
      }
      button.leave {
        background-color: #ef4444;
        color: white;
      }
      .divider {
        width: 1px;
        height: 2rem;
        background-color: #e5e7eb;
        margin: 0 0.5rem;
      }
    `,
  ],
})
export class ControlBarComponent {
  audioOn = input.required<boolean>();
  muted = input.required<boolean>();
  videoOn = input.required<boolean>();
  sharing = input.required<boolean>();
  showChat = input.required<boolean>();
  showParticipants = input.required<boolean>();

  toggleAudio = output<void>();
  toggleVideo = output<void>();
  toggleShare = output<void>();
  leave = output<void>();
  toggleChat = output<void>();
  toggleParticipants = output<void>();
  toggleInfo = output<void>();
}
```

### session-page.component.ts

```typescript
import { Component, signal, effect, OnInit, OnDestroy } from "@angular/core";
import { CommonModule } from "@angular/common";
import { ZoomClientService } from "../../services/zoom-client.service";
import { MediaStreamService } from "../../services/media-stream.service";
import { ParticipantsService } from "../../services/participants.service";
import { ChatService } from "../../services/chat.service";
import { SessionStatusComponent } from "./session-status.component";
import { VideoGridComponent } from "./video-grid.component";
import { ControlBarComponent } from "./control-bar.component";
import type { ViewMode, ConnectionStatus } from "../../models/zoom.types";

@Component({
  selector: "app-session-page",
  standalone: true,
  imports: [
    CommonModule,
    SessionStatusComponent,
    VideoGridComponent,
    ControlBarComponent,
  ],
  template: `
    @if (!zoomClient.isJoined()) {
    <div class="join-form">
      <h1>Join Video Session</h1>
      <!-- Add join form here -->
    </div>
    } @else {
    <div class="session-container">
      <header class="header">
        <app-session-status [status]="status()" />
        <div class="view-controls">
          <button
            [class.active]="viewMode() === 'speaker'"
            (click)="viewMode.set('speaker')"
          >
            Speaker
          </button>
          <button
            [class.active]="viewMode() === 'gallery'"
            (click)="viewMode.set('gallery')"
          >
            Gallery
          </button>
        </div>
      </header>

      <div class="main-content">
        <main class="video-area">
          <app-video-grid
            [participants]="participants.participants()"
            [viewMode]="viewMode()"
            [activeSpeakerId]="participants.activeSpeakerId()"
            [stream]="mediaStream.stream()"
            [currentUserId]="currentUserId()"
            [currentUserVideoOn]="mediaStream.videoOn()"
          />
        </main>

        @if (showParticipants()) {
        <aside class="sidebar">
          <!-- Participant list -->
        </aside>
        } @if (showChat()) {
        <aside class="sidebar">
          <!-- Chat panel -->
        </aside>
        }
      </div>

      <app-control-bar
        [audioOn]="mediaStream.audioOn()"
        [muted]="mediaStream.muted()"
        [videoOn]="mediaStream.videoOn()"
        [sharing]="mediaStream.sharing()"
        [showChat]="showChat()"
        [showParticipants]="showParticipants()"
        (toggleAudio)="handleToggleAudio()"
        (toggleVideo)="handleToggleVideo()"
        (toggleShare)="handleToggleScreenShare()"
        (leave)="handleLeave()"
        (toggleChat)="showChat.update(v => !v)"
        (toggleParticipants)="showParticipants.update(v => !v)"
        (toggleInfo)="showInfo.set(true)"
      />
    </div>
    }
  `,
  styles: [
    `
      .session-container {
        display: flex;
        flex-direction: column;
        height: 100vh;
        background-color: #f9fafb;
      }
      .header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 0.5rem 1rem;
        border-bottom: 1px solid #e5e7eb;
        background-color: white;
      }
      .view-controls {
        display: flex;
        gap: 0.25rem;
      }
      .view-controls button {
        padding: 0.25rem 0.75rem;
        border: none;
        border-radius: 0.375rem;
        background-color: #f3f4f6;
        cursor: pointer;
      }
      .view-controls button.active {
        background-color: #3b82f6;
        color: white;
      }
      .main-content {
        display: flex;
        flex: 1;
        overflow: hidden;
      }
      .video-area {
        flex: 1;
        overflow: hidden;
      }
      .sidebar {
        width: 16rem;
        border-left: 1px solid #e5e7eb;
        background-color: white;
        overflow: hidden;
      }
    `,
  ],
})
export class SessionPageComponent implements OnInit, OnDestroy {
  viewMode = signal<ViewMode>("gallery");
  showChat = signal(false);
  showParticipants = signal(false);
  showInfo = signal(false);
  status = signal<ConnectionStatus>("Disconnected");
  error = signal<string | null>(null);

  currentUserId = signal<number | undefined>(undefined);

  constructor(
    public zoomClient: ZoomClientService,
    public mediaStream: MediaStreamService,
    public participants: ParticipantsService,
    public chat: ChatService
  ) {
    effect(() => {
      if (
        this.zoomClient.isJoined() &&
        this.zoomClient.client() &&
        !this.mediaStream.stream()
      ) {
        this.mediaStream.initStream();
      }
    });

    effect(() => {
      const client = this.zoomClient.client();
      if (client) {
        const user = client.getCurrentUserInfo();
        this.currentUserId.set(user?.userId);
      }
    });
  }

  ngOnInit() {
    this.zoomClient.getClient();
  }

  ngOnDestroy() {
    this.zoomClient.destroy();
  }

  async handleToggleAudio() {
    try {
      this.error.set(null);
      await this.mediaStream.toggleAudio();
    } catch (err) {
      this.error.set(err instanceof Error ? err.message : "Audio error");
      setTimeout(() => this.error.set(null), 3000);
    }
  }

  async handleToggleVideo() {
    try {
      this.error.set(null);
      await this.mediaStream.toggleVideo();
    } catch (err) {
      this.error.set(err instanceof Error ? err.message : "Video error");
      setTimeout(() => this.error.set(null), 3000);
    }
  }

  async handleToggleScreenShare() {
    try {
      this.error.set(null);
      await this.mediaStream.toggleScreenShare();
    } catch (err) {
      this.error.set(err instanceof Error ? err.message : "Screen share error");
      setTimeout(() => this.error.set(null), 3000);
    }
  }

  async handleLeave() {
    this.chat.clearMessages();
    await this.zoomClient.leave();
  }
}
```

## App Component

```typescript
// src/app/app.component.ts
import { Component } from "@angular/core";
import { SessionPageComponent } from "./components/video-session/session-page.component";

@Component({
  selector: "app-root",
  standalone: true,
  imports: [SessionPageComponent],
  template: "<app-session-page />",
})
export class AppComponent {}
```

## Key Angular-Specific Features

1. **Standalone Components** - Modern Angular 19 pattern without NgModules
2. **Signals** - Angular's new reactive primitive for state management
3. **Effect** - Reactive side effects for event listener setup
4. **Services** - Singleton services with dependency injection
5. **Input/Output** - Type-safe component communication
6. **CUSTOM_ELEMENTS_SCHEMA** - Support for Zoom's custom elements
7. **Lifecycle Hooks** - OnInit, OnDestroy for setup/cleanup

## Best Practices

1. Use **signals** for reactive state instead of RxJS where possible
2. Use **effect()** to set up event listeners when signals change
3. Leverage **dependency injection** for services
4. Use **standalone components** for better tree-shaking
5. Implement **OnDestroy** for proper cleanup
6. Use **input()** and **output()** for component props and events
7. Add **CUSTOM_ELEMENTS_SCHEMA** for video-player-container support

## Key Differences from React/Vue

1. **Services vs Hooks/Composables** - Angular uses injectable services
2. **Signals** - Angular's reactivity system (similar to Vue refs)
3. **Effect** - Replaces useEffect/watch for side effects
4. **Dependency Injection** - Built-in DI system
5. **Decorators** - @Component, @Injectable for metadata
6. **Template Syntax** - Angular's template language with @if, @for
7. **Standalone** - No need for NgModules in modern Angular

## Additional Configuration

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "useDefineForClassFields": false,
    "module": "ES2022",
    "lib": ["ES2022", "DOM"],
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true
  }
}
```

This guide provides a complete Angular implementation following modern Angular 19 patterns with standalone components and signals!
