# Svelte Implementation Guide

Complete Svelte 5 + Vite 7 + TypeScript + Runes implementation for Zoom Video SDK.

## Prerequisites

- **Bun** (recommended) or npm/yarn
- **Node.js** 24+
- Basic knowledge of Svelte 5 runes and TypeScript

## Project Setup

### 1. Create Vite Project

```bash
# Using Bun (recommended)
bun create vite my-zoom-app-svelte --template svelte-ts
cd my-zoom-app-svelte

# Or using npm
npm create vite@latest my-zoom-app-svelte -- --template svelte-ts
cd my-zoom-app-svelte
```

### 2. Install Dependencies

```bash
# Core dependencies
bun add @zoom/videosdk

# Development tools
bun add -D @biomejs/biome

# UI components (optional - shadcn-svelte)
bunx shadcn-svelte@latest init
bunx shadcn-svelte@latest add button card avatar badge dialog input scroll-area

# Initialize Biome for linting/formatting
bunx biome init
```

### 3. Additional UI Icons

```bash
# Install lucide-svelte for icons
bun add lucide-svelte
```

## CRITICAL: Vite Configuration

**You MUST configure Vite with COOP/COEP headers** for SharedArrayBuffer support:

```typescript
// vite.config.ts
import { sveltekit } from "@sveltejs/kit/vite";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [sveltekit()],
  server: {
    headers: {
      // required for SharedArrayBuffer for gallery view if not use video_webrtc_mode
      "Cross-Origin-Opener-Policy": "same-origin",
      "Cross-Origin-Embedder-Policy": "require-corp",
    },
  },
});
```

## CRITICAL: TypeScript Custom Elements

```typescript
// src/app.d.ts
declare namespace svelteHTML {
  interface IntrinsicElements {
    "video-player-container": {
      class?: string;
      style?: string;
    };
    "video-player": {
      class?: string;
      style?: string;
    };
  }
}
```

## Project Structure

```
src/
├── lib/
│   ├── components/
│   │   ├── video-session/
│   │   │   ├── SessionStatus.svelte
│   │   │   ├── VideoGrid.svelte
│   │   │   ├── VideoTile.svelte
│   │   │   ├── ParticipantList.svelte
│   │   │   ├── ChatPanel.svelte
│   │   │   ├── ControlBar.svelte
│   │   │   ├── SessionInfo.svelte
│   │   │   └── JoinForm.svelte
│   │   └── ui/
│   ├── stores/
│   │   ├── zoom-client.svelte.ts
│   │   ├── media-stream.svelte.ts
│   │   ├── participants.svelte.ts
│   │   └── chat.svelte.ts
│   └── types/
│       └── zoom.ts
└── routes/
    └── +page.svelte
```

## Types

```typescript
// src/lib/types/zoom.ts
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

## Svelte 5 Stores (with Runes)

### zoom-client.svelte.ts

```typescript
import ZoomVideo, { type VideoClient } from "@zoom/videosdk";

class ZoomClientStore {
  private clientInstance: typeof VideoClient | null = $state(null);

  client = $state<typeof VideoClient | null>(null);
  isInitialized = $state(false);
  isJoined = $state(false);

  getClient(): typeof VideoClient {
    if (!this.clientInstance) {
      this.clientInstance = ZoomVideo.createClient();
      this.client = this.clientInstance;
    }
    return this.clientInstance;
  }

  async init(): Promise<typeof VideoClient> {
    const zoomClient = this.getClient();
    await zoomClient.init("en-US", "Global", {
      patchJsMedia: true,
      leaveOnPageUnload: true,
    });
    this.isInitialized = true;

    zoomClient.on("connection-change", (payload) => {
      if (payload.state === "Closed" || payload.state === "Fail") {
        this.isJoined = false;
      }
    });

    return zoomClient;
  }

  async join(
    sessionName: string,
    token: string,
    userName: string,
    password?: string
  ): Promise<typeof VideoClient> {
    const zoomClient = this.getClient();
    if (!this.isInitialized) {
      await this.init();
    }
    await zoomClient.join(sessionName, token, userName, password);
    this.isJoined = true;
    this.client = zoomClient;
    return zoomClient;
  }

  async leave(endSession = false): Promise<void> {
    if (this.clientInstance) {
      await this.clientInstance.leave(endSession);
      this.isJoined = false;
    }
  }

  destroy(): void {
    if (this.clientInstance) {
      ZoomVideo.destroyClient();
      this.clientInstance = null;
      this.client = null;
      this.isInitialized = false;
      this.isJoined = false;
    }
  }
}

export const zoomClient = new ZoomClientStore();
```

### media-stream.svelte.ts

```typescript
import type { Stream } from "@zoom/videosdk";
import { zoomClient } from "./zoom-client.svelte";

class MediaStreamStore {
  stream = $state<typeof Stream | null>(null);
  audioOn = $state(false);
  videoOn = $state(false);
  sharing = $state(false);
  muted = $state(true);

  constructor() {
    $effect(() => {
      const client = zoomClient.client;
      if (!client) return;

      const audioHandler = (payload: { action: string }) => {
        if (payload.action === "join") {
          this.audioOn = true;
          this.muted = true;
        } else if (payload.action === "leave") {
          this.audioOn = false;
        } else if (payload.action === "muted") {
          this.muted = true;
        } else if (payload.action === "unmuted") {
          this.muted = false;
        }
      };

      // CRITICAL: Sync video state with SDK events
      const videoActiveHandler = (payload: {
        state: string;
        userId: number;
      }) => {
        const currentUser = client.getCurrentUserInfo();
        if (currentUser && payload.userId === currentUser.userId) {
          this.videoOn = payload.state === "Active";
        }
      };

      const shareHandler = (payload: { state: string; userId: number }) => {
        const currentUser = client.getCurrentUserInfo();
        if (currentUser && payload.userId === currentUser.userId) {
          this.sharing = payload.state === "Active";
        }
      };

      client.on("current-audio-change", audioHandler);
      client.on("video-active-change", videoActiveHandler);
      client.on("active-share-change", shareHandler);

      return () => {
        client.off("current-audio-change", audioHandler);
        client.off("video-active-change", videoActiveHandler);
        client.off("active-share-change", shareHandler);
      };
    });
  }

  initStream(): typeof Stream | null {
    const client = zoomClient.client;
    if (client) {
      const mediaStream = client.getMediaStream();
      this.stream = mediaStream;
      return mediaStream;
    }
    return null;
  }

  async toggleAudio(): Promise<void> {
    if (!this.stream) throw new Error("Stream not initialized");

    if (!this.audioOn) {
      await this.stream.startAudio();
    } else if (this.muted) {
      await this.stream.unmuteAudio();
    } else {
      await this.stream.muteAudio();
    }
  }

  async toggleVideo(): Promise<void> {
    if (!this.stream) throw new Error("Stream not initialized");

    if (this.videoOn) {
      await this.stream.stopVideo();
      this.videoOn = false;
    } else {
      await this.stream.startVideo();
      this.videoOn = true;
    }
  }

  async toggleScreenShare(): Promise<void> {
    if (!this.stream) throw new Error("Stream not initialized");

    if (this.sharing) {
      await this.stream.stopShareScreen();
      this.sharing = false;
    } else {
      await this.stream.startShareScreen();
      this.sharing = true;
    }
  }
}

export const mediaStream = new MediaStreamStore();
```

### participants.svelte.ts

```typescript
import type { Participant } from "@zoom/videosdk";
import { zoomClient } from "./zoom-client.svelte";

class ParticipantsStore {
  participants = $state<Participant[]>([]);
  activeSpeakerId = $state<number | null>(null);

  constructor() {
    $effect(() => {
      const client = zoomClient.client;
      if (!client) return;

      const updateList = () => {
        this.participants = client.getAllUser();
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
          this.activeSpeakerId = payload.activeSpeaker[0].oderId;
        }
      };
      client.on("active-speaker", speakerHandler);

      return () => {
        client.off("user-added", updateList);
        client.off("user-updated", updateList);
        client.off("user-removed", updateList);
        client.off("video-active-change", updateList);
        client.off("peer-video-state-change", updateList);
        client.off("connection-change", connectionHandler);
        client.off("active-speaker", speakerHandler);
      };
    });

    // Update participants when joined
    $effect(() => {
      if (zoomClient.isJoined && zoomClient.client) {
        setTimeout(() => {
          if (zoomClient.client) {
            this.participants = zoomClient.client.getAllUser();
          }
        }, 100);
      }
    });
  }
}

export const participants = new ParticipantsStore();
```

### chat.svelte.ts

```typescript
import type { ChatMessage } from "$lib/types/zoom";
import { zoomClient } from "./zoom-client.svelte";

class ChatStore {
  messages = $state<ChatMessage[]>([]);

  constructor() {
    $effect(() => {
      const client = zoomClient.client;
      if (!client) return;

      const handler = (payload: ChatMessage) => {
        this.messages = [
          ...this.messages,
          { ...payload, id: crypto.randomUUID() },
        ];
      };
      client.on("chat-on-message", handler);

      return () => {
        client.off("chat-on-message", handler);
      };
    });
  }

  async sendMessage(text: string): Promise<void> {
    const client = zoomClient.client;
    if (!client) return;
    const chatClient = client.getChatClient();
    await chatClient.send(text);
  }

  clearMessages(): void {
    this.messages = [];
  }
}

export const chat = new ChatStore();
```

## Components

### VideoTile.svelte

```svelte
<script lang="ts">
  import type { Participant, Stream } from '@zoom/videosdk'
  import { VideoOff } from 'lucide-svelte'
  import { onMount, onDestroy } from 'svelte'

  let {
    participant,
    stream,
    isLarge = false,
    isSpeaking = false,
    currentUserId,
    currentUserVideoOn = false,
  }: {
    participant: Participant
    stream: typeof Stream | null
    isLarge?: boolean
    isSpeaking?: boolean
    currentUserId?: number
    currentUserVideoOn?: boolean
  } = $props()

  let containerRef: HTMLElement | null = $state(null)

  const isCurrentUser = $derived(currentUserId === participant.userId)
  const isVideoOn = $derived(isCurrentUser ? currentUserVideoOn : participant.bVideoOn)

  async function attachVideo() {
    if (!stream || !containerRef) return

    if (!isVideoOn) {
      try {
        stream.detachVideo(participant.userId)
        containerRef.innerHTML = ''
      } catch {
        // Ignore detach errors
      }
      return
    }

    try {
      const videoElement = await stream.attachVideo(
        participant.userId,
        3 // VideoQuality.Video_720P
      )

      if (videoElement) {
        containerRef.innerHTML = ''
        const el = videoElement as unknown as HTMLElement
        el.style.width = '100%'
        el.style.height = '100%'
        containerRef.appendChild(el)
      }
    } catch (error) {
      console.error('Error attaching video:', error)
    }
  }

  $effect(() => {
    attachVideo()
  })

  onDestroy(() => {
    if (stream) {
      try {
        stream.detachVideo(participant.userId)
      } catch {
        // Ignore detach errors
      }
    }
  })
</script>

<div
  class="relative bg-gray-800 rounded-lg overflow-hidden flex items-center justify-center {isLarge
    ? 'w-full h-full'
    : 'aspect-video'} {isSpeaking ? 'ring-2 ring-green-500' : ''}"
>
  <video-player-container
    bind:this={containerRef}
    class="w-full h-full"
    style="min-height: 100px; display: {isVideoOn ? 'block' : 'none'}"
  />

  {#if !isVideoOn}
    <div class="flex flex-col items-center justify-center gap-2">
      <div
        class="flex items-center justify-center rounded-full bg-gray-600 text-white font-semibold {isLarge
          ? 'h-24 w-24 text-3xl'
          : 'h-16 w-16 text-xl'}"
      >
        {participant.displayName.charAt(0).toUpperCase()}
      </div>
      <VideoOff class="h-5 w-5 text-gray-400" />
    </div>
  {/if}

  <div class="absolute bottom-2 left-2 right-2 flex items-center justify-between">
    <span class="bg-black/70 text-white px-2 py-1 rounded text-xs truncate max-w-[80%]">
      {participant.displayName}{participant.isHost ? ' (Host)' : ''}
    </span>
  </div>
</div>
```

### VideoGrid.svelte

```svelte
<script lang="ts">
  import type { Participant, Stream } from '@zoom/videosdk'
  import type { ViewMode } from '$lib/types/zoom'
  import VideoTile from './VideoTile.svelte'

  let {
    participants,
    viewMode,
    activeSpeakerId,
    stream,
    currentUserId,
    currentUserVideoOn,
  }: {
    participants: Participant[]
    viewMode: ViewMode
    activeSpeakerId: number | null
    stream: typeof Stream | null
    currentUserId?: number
    currentUserVideoOn?: boolean
  } = $props()

  const speaker = $derived(
    participants.find((p) => p.userId === activeSpeakerId) || participants[0]
  )
  const others = $derived(participants.filter((p) => p.userId !== speaker?.userId))

  function getGridCols(count: number): string {
    if (count === 1) return 'grid-cols-1'
    if (count <= 4) return 'grid-cols-2'
    if (count <= 9) return 'grid-cols-3'
    return 'grid-cols-4'
  }
</script>

{#if participants.length === 0}
  <div class="flex items-center justify-center h-full">
    <p class="text-gray-400">Waiting for participants...</p>
  </div>
{:else if viewMode === 'speaker'}
  <div class="relative h-full p-4">
    <VideoTile
      participant={speaker}
      {stream}
      isLarge={true}
      isSpeaking={speaker.userId === activeSpeakerId}
      {currentUserId}
      {currentUserVideoOn}
    />
    {#if others.length > 0}
      <div class="absolute bottom-6 left-6 flex gap-2">
        {#each others.slice(0, 4) as p (p.userId)}
          <div class="w-32">
            <VideoTile
              participant={p}
              {stream}
              isSpeaking={p.userId === activeSpeakerId}
              {currentUserId}
              {currentUserVideoOn}
            />
          </div>
        {/each}
      </div>
    {/if}
  </div>
{:else}
  <div class="grid {getGridCols(participants.length)} gap-2 h-full p-4">
    {#each participants as p (p.userId)}
      <VideoTile
        participant={p}
        {stream}
        isSpeaking={p.userId === activeSpeakerId}
        {currentUserId}
        {currentUserVideoOn}
      />
    {/each}
  </div>
{/if}
```

### SessionStatus.svelte

```svelte
<script lang="ts">
  import type { ConnectionStatus } from '$lib/types/zoom'

  let { status }: { status: ConnectionStatus } = $props()

  const variants: Record<ConnectionStatus, string> = {
    Connected: 'bg-green-500 text-white',
    Connecting: 'bg-yellow-500 animate-pulse',
    Reconnecting: 'bg-yellow-500 animate-pulse',
    Disconnected: 'bg-gray-500 text-white',
    Failed: 'bg-red-500 text-white',
  }
</script>

<span class="px-3 py-1 rounded-md text-sm font-medium {variants[status]}">
  {status}
</span>
```

### ControlBar.svelte

```svelte
<script lang="ts">
  import { Mic, MicOff, Video, VideoOff, Monitor, Users, MessageSquare, Info, LogOut } from 'lucide-svelte'

  let {
    audioOn,
    muted,
    videoOn,
    sharing,
    showChat,
    showParticipants,
    onToggleAudio,
    onToggleVideo,
    onToggleShare,
    onLeave,
    onToggleChat,
    onToggleParticipants,
    onToggleInfo,
  }: {
    audioOn: boolean
    muted: boolean
    videoOn: boolean
    sharing: boolean
    showChat: boolean
    showParticipants: boolean
    onToggleAudio: () => void
    onToggleVideo: () => void
    onToggleShare: () => void
    onLeave: () => void
    onToggleChat: () => void
    onToggleParticipants: () => void
    onToggleInfo: () => void
  } = $props()
</script>

<div class="flex items-center justify-center gap-2 p-4 border-t bg-white">
  <button
    class="p-3 rounded-lg {audioOn && !muted ? 'bg-blue-500 text-white' : 'bg-gray-200'}"
    onclick={onToggleAudio}
  >
    {#if !audioOn || muted}
      <MicOff class="h-5 w-5" />
    {:else}
      <Mic class="h-5 w-5" />
    {/if}
  </button>

  <button
    class="p-3 rounded-lg {videoOn ? 'bg-blue-500 text-white' : 'bg-gray-200'}"
    onclick={onToggleVideo}
  >
    {#if videoOn}
      <Video class="h-5 w-5" />
    {:else}
      <VideoOff class="h-5 w-5" />
    {/if}
  </button>

  <button
    class="p-3 rounded-lg {sharing ? 'bg-blue-500 text-white' : 'bg-gray-200'}"
    onclick={onToggleShare}
  >
    <Monitor class="h-5 w-5" />
  </button>

  <div class="w-px h-8 bg-gray-300 mx-2" />

  <button
    class="p-3 rounded-lg {showParticipants ? 'bg-blue-500 text-white' : 'bg-gray-200'}"
    onclick={onToggleParticipants}
  >
    <Users class="h-5 w-5" />
  </button>

  <button
    class="p-3 rounded-lg {showChat ? 'bg-blue-500 text-white' : 'bg-gray-200'}"
    onclick={onToggleChat}
  >
    <MessageSquare class="h-5 w-5" />
  </button>

  <button class="p-3 rounded-lg bg-gray-200" onclick={onToggleInfo}>
    <Info class="h-5 w-5" />
  </button>

  <div class="w-px h-8 bg-gray-300 mx-2" />

  <button class="px-4 py-2 rounded-lg bg-red-500 text-white flex items-center gap-2" onclick={onLeave}>
    <LogOut class="h-4 w-4" />
    Leave
  </button>
</div>
```

### SessionPage.svelte

```svelte
<script lang="ts">
  import { onMount, onDestroy } from 'svelte'
  import { zoomClient } from '$lib/stores/zoom-client.svelte'
  import { mediaStream } from '$lib/stores/media-stream.svelte'
  import { participants } from '$lib/stores/participants.svelte'
  import { chat } from '$lib/stores/chat.svelte'
  import type { ViewMode, ConnectionStatus } from '$lib/types/zoom'
  import SessionStatus from './SessionStatus.svelte'
  import VideoGrid from './VideoGrid.svelte'
  import ControlBar from './ControlBar.svelte'

  let viewMode = $state<ViewMode>('gallery')
  let showChat = $state(false)
  let showParticipants = $state(false)
  let showInfo = $state(false)
  let status = $state<ConnectionStatus>('Disconnected')
  let error = $state<string | null>(null)

  const currentUserId = $derived(zoomClient.client?.getCurrentUserInfo()?.userId)

  $effect(() => {
    if (zoomClient.isJoined && zoomClient.client && !mediaStream.stream) {
      mediaStream.initStream()
    }
  })

  async function handleToggleAudio() {
    try {
      error = null
      await mediaStream.toggleAudio()
    } catch (err) {
      error = err instanceof Error ? err.message : 'Audio error'
      setTimeout(() => (error = null), 3000)
    }
  }

  async function handleToggleVideo() {
    try {
      error = null
      await mediaStream.toggleVideo()
    } catch (err) {
      error = err instanceof Error ? err.message : 'Video error'
      setTimeout(() => (error = null), 3000)
    }
  }

  async function handleToggleScreenShare() {
    try {
      error = null
      await mediaStream.toggleScreenShare()
    } catch (err) {
      error = err instanceof Error ? err.message : 'Screen share error'
      setTimeout(() => (error = null), 3000)
    }
  }

  async function handleLeave() {
    chat.clearMessages()
    await zoomClient.leave()
  }

  onMount(() => {
    zoomClient.getClient()
  })

  onDestroy(() => {
    zoomClient.destroy()
  })
</script>

{#if !zoomClient.isJoined}
  <div class="min-h-screen flex items-center justify-center">
    <h1>Join Form (Add JoinForm component here)</h1>
  </div>
{:else}
  <div class="flex flex-col h-screen bg-gray-50">
    <header class="flex items-center justify-between px-4 py-2 border-b bg-white">
      <SessionStatus {status} />
      <div class="flex gap-1">
        <button
          class="px-3 py-1 rounded text-sm {viewMode === 'speaker' ? 'bg-blue-500 text-white' : 'bg-gray-200'}"
          onclick={() => (viewMode = 'speaker')}
        >
          Speaker
        </button>
        <button
          class="px-3 py-1 rounded text-sm {viewMode === 'gallery' ? 'bg-blue-500 text-white' : 'bg-gray-200'}"
          onclick={() => (viewMode = 'gallery')}
        >
          Gallery
        </button>
      </div>
    </header>

    <div class="flex flex-1 overflow-hidden">
      <main class="flex-1 overflow-hidden">
        <VideoGrid
          participants={participants.participants}
          {viewMode}
          activeSpeakerId={participants.activeSpeakerId}
          stream={mediaStream.stream}
          {currentUserId}
          currentUserVideoOn={mediaStream.videoOn}
        />
      </main>

      {#if showParticipants}
        <aside class="w-64 border-l bg-white overflow-hidden">
          <!-- Participant list -->
        </aside>
      {/if}

      {#if showChat}
        <aside class="w-80 border-l bg-white overflow-hidden">
          <!-- Chat panel -->
        </aside>
      {/if}
    </div>

    <ControlBar
      audioOn={mediaStream.audioOn}
      muted={mediaStream.muted}
      videoOn={mediaStream.videoOn}
      sharing={mediaStream.sharing}
      {showChat}
      {showParticipants}
      onToggleAudio={handleToggleAudio}
      onToggleVideo={handleToggleVideo}
      onToggleShare={handleToggleScreenShare}
      onLeave={handleLeave}
      onToggleChat={() => (showChat = !showChat)}
      onToggleParticipants={() => (showParticipants = !showParticipants)}
      onToggleInfo={() => (showInfo = true)}
    />

    {#if error}
      <div class="fixed bottom-20 left-1/2 -translate-x-1/2 bg-red-500 text-white px-4 py-2 rounded-md z-50">
        {error}
      </div>
    {/if}
  </div>
{/if}
```

## Key Svelte 5 Features

1. **Runes** - Modern Svelte 5 reactivity with `$state`, `$derived`, `$effect`, `$props`
2. **Class-based Stores** - Using classes with runes for global state management
3. **Automatic Reactivity** - No need for manual subscriptions
4. **Effect Cleanup** - Return cleanup functions from `$effect()`
5. **Derived State** - Use `$derived` for computed values
6. **Props Destructuring** - Clean component API with `$props()`

## Best Practices

1. Use **$state** for reactive state in stores
2. Use **$effect** for side effects and event listeners
3. Use **$derived** for computed values
4. Return cleanup functions from **$effect** for proper teardown
5. Use **class-based stores** for complex state management
6. Leverage **TypeScript** for type safety
7. Use **bind:this** for DOM element references

## Key Differences from React/Vue/Angular

1. **Runes** - Svelte 5's new reactivity primitives
2. **No Virtual DOM** - Compiles to efficient vanilla JavaScript
3. **Class-based Stores** - Alternative to hooks/composables/services
4. **Effect Cleanup** - Return functions from $effect()
5. **Derived State** - $derived instead of computed/useMemo
6. **Template Syntax** - Svelte's concise template language
7. **Compiler-based** - Reactivity at compile time, not runtime

## Additional Configuration

### tailwind.config.js

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: ["./src/**/*.{html,js,svelte,ts}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

### svelte.config.js

```javascript
import adapter from "@sveltejs/adapter-auto";
import { vitePreprocess } from "@sveltejs/vite-plugin-svelte";

/** @type {import('@sveltejs/kit').Config} */
const config = {
  preprocess: vitePreprocess(),
  kit: {
    adapter: adapter(),
  },
};

export default config;
```

This guide provides a complete Svelte 5 implementation using modern runes and best practices!
