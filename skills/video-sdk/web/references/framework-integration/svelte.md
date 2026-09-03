# Svelte Integration Guide

Focused patterns for integrating `@zoom/videosdk` into a Svelte 5 app (runes +
TypeScript). The SDK concepts are the same across frameworks; this guide shows
the **Svelte-specific** wiring (class-based rune stores, `$state`, `$effect`,
`bind:this`).

> **Read `react.md` first for the canonical SDK patterns.** The SDK-level
> decisions are framework-agnostic and are documented in depth there: holding one
> `createClient()` instance, the init→join lifecycle, the full
> `connection-change` / `ConnectionState` handling (including failover), the
> declarative `video-player` attach API (`attachVideo(userId, quality, element)`),
> receiving screen share with `attachShareView`, and annotation. This guide maps
> those patterns onto Svelte 5 runes and keeps only what is Svelte-specific.

## Architecture at a Glance

- Hold the single `VideoClient` (and, after join, the `MediaStream`) in a
  **class-based rune store** (`*.svelte.ts`) — the Svelte equivalent of React
  Context.
- Use `$state` for reactive fields and `$effect` for subscriptions; **return a
  cleanup function** from `$effect` to unsubscribe.
- Use `bind:this` to capture the `<video-player>` element for `attachVideo`.

## 1. Setup

### Install

```bash
npm install @zoom/videosdk
```

### Vite Configuration (COOP/COEP headers are optional)

Gallery view and virtual background work **without** `SharedArrayBuffer`. The
cross-origin-isolation headers below are **optional**: when present they enable
`SharedArrayBuffer` for better performance; when absent the SDK falls back
automatically.

```typescript
// vite.config.ts
import { sveltekit } from "@sveltejs/kit/vite";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [sveltekit()],
  server: {
    headers: {
      // Optional: enables SharedArrayBuffer for better performance.
      "Cross-Origin-Opener-Policy": "same-origin",
      "Cross-Origin-Embedder-Policy": "require-corp",
    },
  },
});
```

### Type the custom elements (Svelte-specific)

`<video-player-container>` and `<video-player>` are custom elements. Declare them
so Svelte's type checker accepts them:

```typescript
// src/app.d.ts
declare namespace svelteHTML {
  interface IntrinsicElements {
    "video-player-container": { class?: string; style?: string };
    "video-player": { class?: string; style?: string };
  }
}
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

## 2. Client store (create once)

This rune store is the Svelte analog of React's `ZoomContext` + `ZoomMediaContext`
+ the App init/join effect. It creates the client once, joins, and exposes
reactive state. Handle **every** `ConnectionState` — failover/reconnect must not
look like a failure.

```typescript
// src/lib/stores/zoom-client.svelte.ts
import ZoomVideo, {
  ConnectionState,
  ReconnectReason,
  type VideoClient,
  type Stream,
} from "@zoom/videosdk";

export type SessionStatus = "connecting" | "connected" | "closed";

class ZoomClientStore {
  private instance: typeof VideoClient | null = null;

  client = $state<typeof VideoClient | null>(null);
  mediaStream = $state<typeof Stream | null>(null);
  status = $state<SessionStatus>("connecting");

  private getClient(): typeof VideoClient {
    if (!this.instance) {
      this.instance = ZoomVideo.createClient();
      this.client = this.instance;
    }
    return this.instance;
  }

  async join(topic: string, token: string, userName: string, password = ""): Promise<void> {
    const client = this.getClient();
    client.on("connection-change", this.onConnectionChange);
    await client.init("en-US", "Global", { patchJsMedia: true });
    try {
      await client.join(topic, token, userName, password);
      this.mediaStream = client.getMediaStream();
      this.status = "connected";
    } catch (e) {
      this.status = "closed";
      throw e;
    }
  }

  private onConnectionChange = (payload: any) => {
    switch (payload.state) {
      case ConnectionState.Connected:
        this.status = "connected";
        break;
      case ConnectionState.Reconnecting:
        // Failover or main-session/subsession move; show a transient UI,
        // do NOT tear down the session.
        if (payload.reason === ReconnectReason.Failover) {
          /* SDK is recovering */
        }
        this.status = "connecting";
        break;
      case ConnectionState.Closed:
      case ConnectionState.Fail:
        this.status = "closed";
        break;
    }
  };

  async leave(end = false): Promise<void> {
    await this.instance?.leave(end);
  }

  destroy(): void {
    if (this.instance) {
      this.instance.off("connection-change", this.onConnectionChange);
      ZoomVideo.destroyClient();
      this.instance = null;
      this.client = null;
      this.mediaStream = null;
    }
  }
}

export const zoomClient = new ZoomClientStore();
```

Kick it off from a component:

```svelte
<script lang="ts">
  import { onDestroy } from "svelte";
  import { zoomClient } from "$lib/stores/zoom-client.svelte";
  import VideoSession from "$lib/components/VideoSession.svelte";

  zoomClient.join("demo", "<JWT>", "alice");
  onDestroy(() => zoomClient.destroy());
</script>

{#if zoomClient.status === "connected"}
  <VideoSession />
{:else}
  <p>Joining…</p>
{/if}
```

## 3. Participant synchronization

Mirror React's `useParticipantsChange`: refresh from `getAllUser()` on every
membership/state change. Return a cleanup function from `$effect` to unsubscribe.

```typescript
// src/lib/stores/participants.svelte.ts
import type { Participant } from "@zoom/videosdk";
import { zoomClient } from "./zoom-client.svelte";

class ParticipantsStore {
  participants = $state<Participant[]>([]);
  activeSpeakerId = $state(0);

  constructor() {
    $effect(() => {
      const client = zoomClient.client;
      if (!client) return;

      const refresh = () => (this.participants = client.getAllUser());
      // active-speaker payload is an array; read userId (not "oderId").
      const onActiveSpeaker = (payload: Array<{ userId: number }>) => {
        if (payload.length) this.activeSpeakerId = payload[0].userId;
      };

      client.on("user-added", refresh);
      client.on("user-removed", refresh);
      client.on("user-updated", refresh);
      client.on("active-speaker", onActiveSpeaker);
      refresh();

      return () => {
        client.off("user-added", refresh);
        client.off("user-removed", refresh);
        client.off("user-updated", refresh);
        client.off("active-speaker", onActiveSpeaker);
      };
    });
  }
}

export const participants = new ParticipantsStore();
```

## 4. Rendering video (declarative `video-player`)

Same rule as React: render a `<video-player>` only when `bVideoOn` is true,
capture it with `bind:this`, and pass it to `attachVideo(userId, quality,
element)`. A per-tile component with a `$effect` re-attaches whenever the element
appears or the stream becomes available.

```svelte
<!-- src/lib/components/VideoTile.svelte -->
<script lang="ts">
  import { VideoQuality, type Stream } from "@zoom/videosdk";
  import { onDestroy } from "svelte";

  let {
    userId,
    videoOn,
    name = "",
    stream,
  }: {
    userId: number;
    videoOn: boolean;
    name?: string;
    stream: typeof Stream | null;
  } = $props();

  let playerEl = $state<HTMLElement | null>(null);

  $effect(() => {
    if (!stream) return;
    if (videoOn && playerEl) {
      stream.attachVideo(userId, VideoQuality.Video_720P, playerEl);
    } else {
      stream.detachVideo(userId);
    }
  });

  onDestroy(() => stream?.detachVideo(userId));
</script>

<video-player-container class="tile">
  {#if videoOn}
    <video-player bind:this={playerEl} class="video-player"></video-player>
  {:else}
    <div class="avatar">{name}</div>
  {/if}
</video-player-container>
```

Render the grid by iterating participants:

```svelte
<script lang="ts">
  import { participants } from "$lib/stores/participants.svelte";
  import { zoomClient } from "$lib/stores/zoom-client.svelte";
  import VideoTile from "./VideoTile.svelte";
</script>

<div class="video-grid">
  {#each participants.participants as user (user.userId)}
    <VideoTile
      userId={user.userId}
      videoOn={user.bVideoOn}
      name={user.displayName}
      stream={zoomClient.mediaStream}
    />
  {/each}
</div>
```

> A separate `<video-player-container>` per tile is fine. The only hard rule:
> the share-view container must be **separate** from video containers.

## 5. Layouts, screen share, annotation, controls

These are framework-agnostic — implement them as described in `react.md`,
translating React state to `$state` and React effects to `$effect`:

- **Layouts (1:1 / speaker / gallery)**: `$derived` layout on top of the tile
  component; track the active speaker via the `active-speaker` event
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

## Svelte-specific Pitfalls

- **Missing the `svelteHTML` custom-element declaration** → type errors on
  `<video-player-container>` / `<video-player>`.
- **Not returning a cleanup from `$effect`** → duplicate `client.on(...)`
  subscriptions and leaks. Always return the `off(...)` teardown.
- **`active-speaker` payload is an array** — read `payload[0].userId`.
- **Don't tear down on `Reconnecting`** — failover/subsession moves emit it.
- **`detachVideo` on destroy** — clean up each tile in `onDestroy`.

## Official references

- The full SDK call sequences (used by every framework) live in the official
  React sample: **<https://github.com/zoom/videosdk-web-sample>**. See the
  Reference Map in `react.md` to locate each concern.
- Framework setup notes: `references/frameworks.md`.
