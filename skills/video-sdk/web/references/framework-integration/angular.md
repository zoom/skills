# Angular Integration Guide

Focused patterns for integrating `@zoom/videosdk` into an Angular app
(standalone components + signals). The SDK concepts are the same across
frameworks; this guide shows the **Angular-specific** wiring and the two things
Angular requires that other frameworks do not: the custom-elements schema and
the zone configuration.

> **Read `react.md` first for the canonical SDK patterns.** The SDK-level
> decisions are framework-agnostic and are documented in depth there: holding one
> `createClient()` instance, the init→join lifecycle, the full
> `connection-change` / `ConnectionState` handling (including failover), the
> declarative `video-player` attach API (`attachVideo(userId, quality, element)`),
> receiving screen share with `attachShareView`, and annotation. This guide maps
> those patterns onto Angular (injectable services, signals,
> `NgZone.runOutsideAngular`) and keeps only what is Angular-specific.

## Architecture at a Glance

- Hold the single `VideoClient` (and, after join, the `MediaStream`) in a
  **root-provided injectable service** — the Angular equivalent of React Context.
- Use **signals** for reactive state; components read them directly in templates.
- **Run all SDK work outside the Angular zone** (`NgZone.runOutsideAngular`) and
  **push state back inside the zone** (`ngZone.run`) so change detection fires
  exactly once per update instead of on every SDK animation frame.

## 1. Setup

### Install

```bash
npm install @zoom/videosdk
```

### Vite Configuration (COOP/COEP headers are optional)

Gallery view and virtual background work **without** `SharedArrayBuffer`. If you
run Angular on Vite (Angular 17+), the cross-origin-isolation headers below are
**optional**: when present they enable `SharedArrayBuffer` for better
performance; when absent the SDK falls back automatically.

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import angular from "@analogjs/vite-plugin-angular";

export default defineConfig({
  plugins: [angular()],
  server: {
    headers: {
      // Optional: enables SharedArrayBuffer for better performance.
      "Cross-Origin-Opener-Policy": "same-origin",
      "Cross-Origin-Embedder-Policy": "require-corp",
    },
  },
});
```

### Custom-elements schema (Angular-specific, required)

`<video-player-container>` and `<video-player>` are custom elements. Angular
rejects unknown elements unless you add `CUSTOM_ELEMENTS_SCHEMA` to every
standalone component (or app config) that renders them.

```typescript
import { CUSTOM_ELEMENTS_SCHEMA, Component } from "@angular/core";

@Component({
  // ...
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
})
export class VideoTileComponent {}
```

### Custom-element CSS

The custom elements default to `display: inline` with zero size — without CSS,
`attachVideo()` succeeds but renders nothing. The container needs a resolved
height and each `video-player` should keep the video aspect ratio. See
`references/video/video.md`; minimal version:

```css
video-player-container {
  width: 100%;
  height: 600px; /* container must have a resolved height */
}
video-player {
  display: block;
  width: 100%;
  height: auto;
  aspect-ratio: 16 / 9;
}
```

## 2. Angular zone configuration (Angular-specific, required)

zone.js patches `requestAnimationFrame` and event listeners, which the SDK uses
heavily for rendering. Left as-is, this triggers excessive change detection and
UI jank. Unpatch them and run SDK work outside the zone, following the official
`references/frameworks.md` Angular steps:

**a. Create `src/zone-flags.ts`:**

```typescript
// src/zone-flags.ts
(window as any).__Zone_disable_requestAnimationFrame = true;
(window as any).__zone_symbol__UNPATCHED_EVENTS = ["message"];
```

**b. Add it to `angular.json` polyfills, before `zone.js`:**

```json
"polyfills": ["src/zone-flags.ts", "zone.js"]
```

**c. Include it in `tsconfig.app.json`:**

```json
"include": ["src/**/*.d.ts", "src/zone-flags.ts"]
```

**d. Run every SDK call inside `ngZone.runOutsideAngular(...)`** (see the service
below). This is the single most important Angular-specific rule.

## 3. Client service (create once, run outside the zone)

This service is the Angular analog of React's `ZoomContext` + `ZoomMediaContext`
+ the App init/join effect. It creates the client once, joins outside the zone,
and exposes signals. SDK event callbacks fire outside the Angular zone, so every
state write is wrapped in `ngZone.run(...)`.

```typescript
// src/app/services/zoom-client.service.ts
import { Injectable, NgZone, signal } from "@angular/core";
import ZoomVideo, {
  ConnectionState,
  ReconnectReason,
  type VideoClient,
  type Stream,
} from "@zoom/videosdk";

export type SessionStatus = "connecting" | "connected" | "closed";

@Injectable({ providedIn: "root" })
export class ZoomClientService {
  private clientInstance: typeof VideoClient | null = null;

  readonly client = signal<typeof VideoClient | null>(null);
  readonly mediaStream = signal<typeof Stream | null>(null);
  readonly status = signal<SessionStatus>("connecting");

  constructor(private ngZone: NgZone) {}

  private getClient(): typeof VideoClient {
    if (!this.clientInstance) {
      this.clientInstance = ZoomVideo.createClient();
      this.client.set(this.clientInstance);
    }
    return this.clientInstance;
  }

  async join(topic: string, token: string, userName: string, password = ""): Promise<void> {
    await this.ngZone.runOutsideAngular(async () => {
      const client = this.getClient();
      client.on("connection-change", this.onConnectionChange);
      await client.init("en-US", "Global", { patchJsMedia: true });
      try {
        await client.join(topic, token, userName, password);
        this.ngZone.run(() => {
          this.mediaStream.set(client.getMediaStream());
          this.status.set("connected");
        });
      } catch (e) {
        this.ngZone.run(() => this.status.set("closed"));
        throw e;
      }
    });
  }

  // Handle every ConnectionState; failover/reconnect must not look like failure.
  private onConnectionChange = (payload: any) => {
    this.ngZone.run(() => {
      switch (payload.state) {
        case ConnectionState.Connected:
          this.status.set("connected");
          break;
        case ConnectionState.Reconnecting:
          // Failover or main-session/subsession move; show a transient UI,
          // do NOT tear down the session.
          if (payload.reason === ReconnectReason.Failover) {
            /* SDK is recovering */
          }
          this.status.set("connecting");
          break;
        case ConnectionState.Closed:
        case ConnectionState.Fail:
          this.status.set("closed");
          break;
      }
    });
  };

  leave(end = false): Promise<void> {
    return this.ngZone.runOutsideAngular(() => this.clientInstance!.leave(end));
  }

  destroy(): void {
    if (this.clientInstance) {
      this.clientInstance.off("connection-change", this.onConnectionChange);
      ZoomVideo.destroyClient();
      this.clientInstance = null;
      this.client.set(null);
      this.mediaStream.set(null);
    }
  }
}
```

Kick it off from the root component:

```typescript
// src/app/app.component.ts
import { Component, OnDestroy, OnInit, inject } from "@angular/core";
import { ZoomClientService } from "./services/zoom-client.service";

@Component({
  selector: "app-root",
  standalone: true,
  template: `
    @if (zoom.status() === "connected") {
      <app-video-session />
    } @else {
      <p>Joining…</p>
    }
  `,
})
export class AppComponent implements OnInit, OnDestroy {
  zoom = inject(ZoomClientService);

  ngOnInit() {
    this.zoom.join("demo", "<JWT>", "alice");
  }
  ngOnDestroy() {
    this.zoom.destroy();
  }
}
```

## 4. Participant synchronization

Mirror React's `useParticipantsChange`: refresh from `getAllUser()` on every
membership/state change. Wrap the signal write in `ngZone.run` because the
callback fires outside the zone.

```typescript
// src/app/services/participants.service.ts
import { Injectable, NgZone, effect, signal } from "@angular/core";
import type { Participant } from "@zoom/videosdk";
import { ZoomClientService } from "./zoom-client.service";

@Injectable({ providedIn: "root" })
export class ParticipantsService {
  readonly participants = signal<Participant[]>([]);
  readonly activeSpeakerId = signal(0);

  constructor(private zoom: ZoomClientService, private ngZone: NgZone) {
    effect((onCleanup) => {
      const client = this.zoom.client();
      if (!client) return;

      const refresh = () => this.ngZone.run(() => this.participants.set(client.getAllUser()));
      // active-speaker payload is an array; read userId (not "oderId").
      const onActiveSpeaker = (payload: Array<{ userId: number }>) => {
        if (payload.length) this.ngZone.run(() => this.activeSpeakerId.set(payload[0].userId));
      };

      client.on("user-added", refresh);
      client.on("user-removed", refresh);
      client.on("user-updated", refresh);
      client.on("active-speaker", onActiveSpeaker);
      refresh();

      onCleanup(() => {
        client.off("user-added", refresh);
        client.off("user-removed", refresh);
        client.off("user-updated", refresh);
        client.off("active-speaker", onActiveSpeaker);
      });
    });
  }
}
```

## 5. Rendering video (declarative `video-player`)

Same rule as React: declare a `<video-player>` only when `bVideoOn` is true,
capture the element, and pass it to `attachVideo(userId, quality, element)`. In
Angular, a per-tile component with a `viewChild` signal + `effect` is the
cleanest expression — the effect re-attaches whenever the element appears or the
stream becomes available.

```typescript
// src/app/components/video-tile.component.ts
import {
  CUSTOM_ELEMENTS_SCHEMA,
  Component,
  ElementRef,
  OnDestroy,
  effect,
  inject,
  input,
  viewChild,
} from "@angular/core";
import { VideoQuality } from "@zoom/videosdk";
import { ZoomClientService } from "../services/zoom-client.service";

@Component({
  selector: "app-video-tile",
  standalone: true,
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
  template: `
    <video-player-container class="tile">
      @if (videoOn()) {
        <video-player #player class="video-player"></video-player>
      } @else {
        <div class="avatar">{{ name() }}</div>
      }
    </video-player-container>
  `,
})
export class VideoTileComponent implements OnDestroy {
  private zoom = inject(ZoomClientService);
  userId = input.required<number>();
  videoOn = input.required<boolean>();
  name = input("");
  private player = viewChild<ElementRef>("player");

  constructor() {
    effect(() => {
      const stream = this.zoom.mediaStream();
      const el = this.player()?.nativeElement;
      if (!stream) return;
      if (this.videoOn() && el) {
        stream.attachVideo(this.userId(), VideoQuality.Video_720P, el);
      } else {
        stream.detachVideo(this.userId());
      }
    });
  }

  ngOnDestroy() {
    this.zoom.mediaStream()?.detachVideo(this.userId());
  }
}
```

Render the grid by iterating participants:

```typescript
template: `
  <div class="video-grid">
    @for (user of participants.participants(); track user.userId) {
      <app-video-tile [userId]="user.userId" [videoOn]="user.bVideoOn" [name]="user.displayName" />
    }
  </div>
`,
```

> A separate `<video-player-container>` per tile is fine. The only hard rule:
> the share-view container must be **separate** from video containers.

## 6. Layouts, screen share, annotation, controls

These are framework-agnostic — implement them exactly as described in `react.md`,
translating React state to signals and React effects to Angular `effect`s:

- **Layouts (1:1 / speaker / gallery)**: plain signal-driven layout on top of the
  tile component. Track the active speaker via the `active-speaker` event
  (array → `payload[0].userId`). See `react.md` §7.
- **Screen share**: send with `startShareScreen(element)` (choose `<video>` vs
  `<canvas>` via `isStartShareScreenWithVideoElement()`); receive a peer's share
  with `attachShareView(userId, element)` / `detachShareView`. Drive state from
  `active-share-change`, `peer-share-state-change`,
  `share-content-dimension-change`, `passively-stop-share`. See `react.md` §8.
- **Annotation**: recompute `mediaStream.canDoAnnotation()` on share/privilege
  changes; toggle with `startAnnotation()` / `stopAnnotation()`. See `react.md` §9.
- **Audio/video controls**: `startAudio()` joins unmuted — call `muteAudio()`
  after; guard toggles against duplicate clicks. Keep UI in sync via
  `current-audio-change` and `video-active-change`. See `react.md` §10.

Remember to wrap any signal write made from an SDK event callback in
`ngZone.run(...)`.

## Angular-specific Pitfalls

- **Missing `CUSTOM_ELEMENTS_SCHEMA`** → template compile error on
  `<video-player-container>` / `<video-player>`.
- **Skipping the zone config** (`zone-flags.ts` + `runOutsideAngular`) → severe
  performance degradation / UI freezing during rendering.
- **Updating signals from SDK callbacks without `ngZone.run`** → state changes
  that don't trigger change detection (stale UI). Conversely, don't run SDK media
  calls inside the zone.
- **`active-speaker` payload is an array** — read `payload[0].userId`.
- **Don't tear down on `Reconnecting`** — failover/subsession moves emit it.
- **Clean up** every `client.on(...)` (use `effect`'s `onCleanup` or
  `ngOnDestroy`) and `detachVideo` / `detachShareView` on teardown.

## Official references

- Angular zone setup and the Video SDK Angular example: `references/frameworks.md`
  (the "Using Angular" section). Also see the dev blog,
  [Build a video conferencing app with the Zoom Video SDK & Angular](https://developers.zoom.us/blog/angular-video-conferencing-app).
- The full SDK call sequences (used by every framework) live in the official
  React sample: **<https://github.com/zoom/videosdk-web-sample>**. See the
  Reference Map in `react.md` to locate each concern.
