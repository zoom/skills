# Vue Integration Guide

Focused patterns for integrating `@zoom/videosdk` into a Vue 3 app (Composition
API + TypeScript). The SDK concepts are the same across frameworks; this guide
shows the **Vue-specific** wiring (composables, `ref`/`shallowRef`, `watch`,
template refs).

> **Read `react.md` first for the canonical SDK patterns.** The SDK-level
> decisions are framework-agnostic and are documented in depth there: holding one
> `createClient()` instance, the init→join lifecycle, the full
> `connection-change` / `ConnectionState` handling (including failover), the
> declarative `video-player` attach API (`attachVideo(userId, quality, element)`),
> receiving screen share with `attachShareView`, and annotation. This guide maps
> those patterns onto Vue reactivity and keeps only what is Vue-specific.

## Architecture at a Glance

- Hold the single `VideoClient` (and, after join, the `MediaStream`) in a
  **composable** that you call once near the root and inject/share downstream —
  the Vue equivalent of React Context.
- Use `shallowRef` for the client/stream (they are large, non-plain objects),
  `ref` for primitives, and `watch` for subscriptions.
- Use a **template ref** to capture the `<video-player>` element for `attachVideo`.

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
import vue from "@vitejs/plugin-vue";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [vue()],
  server: {
    headers: {
      // Optional: enables SharedArrayBuffer for better performance.
      "Cross-Origin-Opener-Policy": "same-origin",
      "Cross-Origin-Embedder-Policy": "require-corp",
    },
  },
});
```

### Type the custom elements (Vue-specific)

`<video-player-container>` and `<video-player>` are custom elements. Declare them
so Vue's template type checker accepts them:

```typescript
// src/types/zoom-elements.d.ts
import type { DefineComponent } from "vue";

declare module "vue" {
  interface GlobalComponents {
    "video-player-container": DefineComponent<{}, {}, any>;
    "video-player": DefineComponent<{}, {}, any>;
  }
}

export {};
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

## 2. Client composable (create once)

This composable is the Vue analog of React's `ZoomContext` + `ZoomMediaContext`
+ the App init/join effect. It creates the client once, joins, and exposes
reactive state. Handle **every** `ConnectionState` — failover/reconnect must not
look like a failure. Call it once near the root and pass the result down (or use
`provide`/`inject`).

```typescript
// src/composables/useZoomClient.ts
import ZoomVideo, {
  ConnectionState,
  ReconnectReason,
  type VideoClient,
  type Stream,
} from "@zoom/videosdk";
import { shallowRef, ref } from "vue";

export type SessionStatus = "connecting" | "connected" | "closed";

export function useZoomClient() {
  const client = shallowRef<typeof VideoClient | null>(null);
  const mediaStream = shallowRef<typeof Stream | null>(null);
  const status = ref<SessionStatus>("connecting");

  const getClient = () => {
    if (!client.value) client.value = ZoomVideo.createClient();
    return client.value;
  };

  const onConnectionChange = (payload: any) => {
    switch (payload.state) {
      case ConnectionState.Connected:
        status.value = "connected";
        break;
      case ConnectionState.Reconnecting:
        // Failover or main-session/subsession move; show a transient UI,
        // do NOT tear down the session.
        if (payload.reason === ReconnectReason.Failover) {
          /* SDK is recovering */
        }
        status.value = "connecting";
        break;
      case ConnectionState.Closed:
      case ConnectionState.Fail:
        status.value = "closed";
        break;
    }
  };

  const join = async (topic: string, token: string, userName: string, password = "") => {
    const c = getClient();
    c.on("connection-change", onConnectionChange);
    await c.init("en-US", "Global", { patchJsMedia: true });
    try {
      await c.join(topic, token, userName, password);
      mediaStream.value = c.getMediaStream();
      status.value = "connected";
    } catch (e) {
      status.value = "closed";
      throw e;
    }
  };

  const leave = (end = false) => client.value?.leave(end);

  const destroy = () => {
    if (client.value) {
      client.value.off("connection-change", onConnectionChange);
      ZoomVideo.destroyClient();
      client.value = null;
      mediaStream.value = null;
    }
  };

  return { client, mediaStream, status, join, leave, destroy };
}
```

Kick it off from the root component:

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { onUnmounted } from "vue";
import { useZoomClient } from "@/composables/useZoomClient";
import VideoSession from "@/components/VideoSession.vue";

const { status, mediaStream, join, destroy } = useZoomClient();
join("demo", "<JWT>", "alice");
onUnmounted(destroy);
</script>

<template>
  <VideoSession v-if="status === 'connected'" :stream="mediaStream" />
  <p v-else>Joining…</p>
</template>
```

## 3. Participant synchronization

Mirror React's `useParticipantsChange`: refresh from `getAllUser()` on every
membership/state change. Clean up listeners on unmount.

```typescript
// src/composables/useParticipants.ts
import type { Participant, VideoClient } from "@zoom/videosdk";
import { onUnmounted, ref, watch, type Ref } from "vue";

export function useParticipants(client: Ref<typeof VideoClient | null>) {
  const participants = ref<Participant[]>([]);
  const activeSpeakerId = ref(0);

  watch(
    client,
    (c, _prev, onCleanup) => {
      if (!c) return;

      const refresh = () => (participants.value = c.getAllUser());
      // active-speaker payload is an array; read userId (not "oderId").
      const onActiveSpeaker = (payload: Array<{ userId: number }>) => {
        if (payload.length) activeSpeakerId.value = payload[0].userId;
      };

      c.on("user-added", refresh);
      c.on("user-removed", refresh);
      c.on("user-updated", refresh);
      c.on("active-speaker", onActiveSpeaker);
      refresh();

      onCleanup(() => {
        c.off("user-added", refresh);
        c.off("user-removed", refresh);
        c.off("user-updated", refresh);
        c.off("active-speaker", onActiveSpeaker);
      });
    },
    { immediate: true }
  );

  return { participants, activeSpeakerId };
}
```

## 4. Rendering video (declarative `video-player`)

Same rule as React: render a `<video-player>` only when `bVideoOn` is true,
capture it with a template ref, and pass it to `attachVideo(userId, quality,
element)`. A per-tile component with a `watch` re-attaches whenever the element
appears or the stream becomes available.

```vue
<!-- src/components/VideoTile.vue -->
<script setup lang="ts">
import { VideoQuality, type Stream } from "@zoom/videosdk";
import { onBeforeUnmount, ref, watch, type PropType } from "vue";

const props = defineProps({
  userId: { type: Number, required: true },
  videoOn: { type: Boolean, required: true },
  name: { type: String, default: "" },
  stream: { type: Object as PropType<typeof Stream | null>, default: null },
});

const playerRef = ref<HTMLElement | null>(null);

watch(
  () => [props.videoOn, props.stream, playerRef.value] as const,
  ([videoOn, stream, el]) => {
    if (!stream) return;
    if (videoOn && el) stream.attachVideo(props.userId, VideoQuality.Video_720P, el);
    else stream.detachVideo(props.userId);
  },
  { immediate: true }
);

onBeforeUnmount(() => props.stream?.detachVideo(props.userId));
</script>

<template>
  <video-player-container class="tile">
    <video-player v-if="videoOn" ref="playerRef" class="video-player" />
    <div v-else class="avatar">{{ name }}</div>
  </video-player-container>
</template>
```

Render the grid by iterating participants:

```vue
<template>
  <div class="video-grid">
    <VideoTile
      v-for="user in participants"
      :key="user.userId"
      :user-id="user.userId"
      :video-on="user.bVideoOn"
      :name="user.displayName"
      :stream="stream"
    />
  </div>
</template>
```

> A separate `<video-player-container>` per tile is fine. The only hard rule:
> the share-view container must be **separate** from video containers.

## 5. Layouts, screen share, annotation, controls

These are framework-agnostic — implement them as described in `react.md`,
translating React state to `ref`/`computed` and React effects to `watch`:

- **Layouts (1:1 / speaker / gallery)**: `computed` layout on top of the tile
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

## Vue-specific Pitfalls

- **Missing the `GlobalComponents` custom-element declaration** → template type
  errors on `<video-player-container>` / `<video-player>`.
- **Using `ref` instead of `shallowRef` for the client/stream** → Vue deeply
  proxies the SDK object, which can break it. Always use `shallowRef`.
- **Forgetting `onCleanup` in `watch`** → duplicate `client.on(...)`
  subscriptions when the client ref changes.
- **`active-speaker` payload is an array** — read `payload[0].userId`.
- **Don't tear down on `Reconnecting`** — failover/subsession moves emit it.
- **`detachVideo` on unmount** — clean up each tile in `onBeforeUnmount`.

## Official references

- The full SDK call sequences (used by every framework) live in the official
  React sample: **<https://github.com/zoom/videosdk-web-sample>**. See the
  Reference Map in `react.md` to locate each concern.
- Framework setup notes: `references/frameworks.md`.
