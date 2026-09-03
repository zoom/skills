# React Integration Guide

Focused, production-grade patterns for integrating `@zoom/videosdk` into a React
app. These patterns are derived from the official Video SDK React sample
(`zoom/videosdk-web-sample`, the `Video` / `VideoAttach` features). Prefer this
architecture over ad-hoc prop drilling.

This guide is a **reference for the SDK-specific decisions** that matter in
React (where to hold the client, how to render `video-player`, how to react to
events). It is not a full app tutorial — wire the UI with whatever component/UI
library the existing project already uses.

## Architecture at a Glance

- Create the `VideoClient` **once** with `ZoomVideo.createClient()` and expose it
  through a React Context (`ZoomContext`). Every component reads the same
  instance with `useContext`.
- After `join()` succeeds, call `client.getMediaStream()` and expose the
  `MediaStream` through a second Context (`ZoomMediaContext`). Media APIs
  (`startAudio`, `attachVideo`, `startShareScreen`, …) live on the stream, so a
  separate context keeps render code clean and avoids "stream is null" guards
  everywhere.
- Treat SDK events as the source of truth. Components subscribe to events and
  reconcile with getters (`getAllUser`, `getCurrentUserInfo`,
  `getShareUserList`).
- All SDK media operations return Promises. Guard against duplicate in-flight
  calls and surface a loading state in the UI.

```
ZoomContext.Provider (client, created once)
└─ <App> (init + join lifecycle, connection-change)
   └─ ZoomMediaContext.Provider (mediaStream, available after join)
      └─ <VideoSession> (layout, controls)
         ├─ video rendering (video-player-container / video-player)
         ├─ screen share (attachShareView)
         └─ annotation
```

## 1. Setup

### Install

```bash
npm install @zoom/videosdk
```

### Vite dev-server headers (COOP/COEP) — optional

Gallery view and virtual background work **without** `SharedArrayBuffer`. Setting
the cross-origin-isolation headers is **optional**: when present they enable
`SharedArrayBuffer`, which the SDK uses for better performance (more concurrent
rendered videos, lower CPU). When absent, the SDK falls back automatically and
everything still functions.

If you want the `SharedArrayBuffer` performance path, set the headers on the dev
server (and replicate them on your production host):

```typescript
// vite.config.ts
import react from "@vitejs/plugin-react";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [react()],
  server: {
    headers: {
      // Optional: enables SharedArrayBuffer for better performance.
      "Cross-Origin-Opener-Policy": "same-origin",
      "Cross-Origin-Embedder-Policy": "require-corp",
    },
  },
});
```

Without cross-origin isolation, pass `enforceMultipleVideos` to `client.init()`
to keep multi-video gallery view enabled, and use the WebRTC video mode where
available. See `references/browser-support.md`.

### Type the custom elements for JSX

`<video-player-container>` and `<video-player>` are custom elements. Declare
them so TSX type-checks. For React 19:

```typescript
// src/types/zoom-elements.d.ts
import type { VideoPlayer, VideoPlayerContainer } from "@zoom/videosdk";
import type { DetailedHTMLProps, HTMLAttributes } from "react";

declare module "react" {
  namespace JSX {
    interface IntrinsicElements {
      "video-player-container": DetailedHTMLProps<
        HTMLAttributes<VideoPlayerContainer>,
        VideoPlayerContainer
      > & { class?: string };
      "video-player": DetailedHTMLProps<
        HTMLAttributes<VideoPlayer>,
        VideoPlayer
      > & { class?: string };
    }
  }
}
```

> For React 18 and earlier, declare the same `IntrinsicElements` inside
> `declare global { namespace JSX { ... } }` instead of `declare module "react"`.

### Custom-element CSS

The custom elements default to `display: inline` with zero size. Without CSS,
`attachVideo()` succeeds but renders nothing (black/empty area, no error). Two
things matter: the container must have a **resolved height**, and each
`video-player` should keep the video's aspect ratio (16:9 on desktop/landscape,
9:16 in portrait on mobile). Mirror the official CSS from
`references/video/video.md`:

```css
/* adjust to your layout */
video-player-container {
  width: 100%;
  height: 600px; /* the container must have an explicit/resolved height */
}

video-player {
  width: 100%;
  height: auto;
  aspect-ratio: 16 / 9;
}
```

For a multi-video gallery, lay the container out with flex (or grid) and size
each player as a fraction of the row:

```css
video-player-container {
  width: 100%;
  height: 100%; /* parent must define a height */
  display: flex !important;
  flex-wrap: wrap;
  align-content: baseline;
}

video-player {
  width: 100%;
  height: auto;
  flex: 0 0 50%; /* two columns */
  aspect-ratio: 16 / 9;
}
```

> If you wrap each `<video-player>` in a layout cell (as in §6 / §7), put the
> `aspect-ratio` on the wrapper element and let the player fill it with
> `display: block; width: 100%; height: 100%; object-fit: contain;`.

### Generic helper hooks

The examples below use two standard, framework-generic React hooks (not
Zoom-specific). They are shown here so the snippets are self-contained — reuse
your project's equivalents or a utility library if you already have them. The one
detail that matters: `usePrevious` must update its ref in an **effect** (after
render) so that during render it still returns the value from the previous
render.

```typescript
// src/hooks/usePrevious.ts
import { useEffect, useRef } from "react";

export function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
}
```

```typescript
// src/hooks/useMount.ts
import { useEffect } from "react";

export function useMount(fn: () => void) {
  // eslint-disable-next-line react-hooks/exhaustive-deps
  useEffect(() => {
    fn();
  }, []);
}
```

## 2. Create the client once and provide it via Context

```typescript
// src/context/zoom-context.ts
import { createContext } from "react";
import type { VideoClient } from "@zoom/videosdk";

export const ZoomContext = createContext<typeof VideoClient>(null as never);
```

```typescript
// src/context/media-context.ts
import { createContext } from "react";
import type { Stream } from "@zoom/videosdk";

interface MediaContextState {
  mediaStream: typeof Stream | null;
}

export const ZoomMediaContext = createContext<MediaContextState>(null as never);
```

```tsx
// src/main.tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import ZoomVideo from "@zoom/videosdk";
import App from "./App";
import { ZoomContext } from "./context/zoom-context";
import "./index.css";

// Create the client exactly once, outside the React tree.
const client = ZoomVideo.createClient();

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <ZoomContext.Provider value={client}>
      <App />
    </ZoomContext.Provider>
  </StrictMode>
);
```

> **StrictMode is fine** with this pattern. The client is created once outside
> the tree (not in an effect), and the init effect below is guarded with a ref,
> so StrictMode's double-invoke in development does not create/destroy duplicate
> clients. Do not call `createClient()` inside a component body or effect.

## 3. Init + join lifecycle (StrictMode-safe)

`init()` must complete before `join()`. Guard the effect with a ref so it runs
its async work only once, then expose the `MediaStream` through context.

```tsx
// src/App.tsx
import { useContext, useEffect, useRef, useState, useMemo } from "react";
import ZoomVideo from "@zoom/videosdk";
import type { Stream } from "@zoom/videosdk";
import { ZoomContext } from "./context/zoom-context";
import { ZoomMediaContext } from "./context/media-context";
import VideoSession from "./VideoSession";

const SESSION = { topic: "demo", token: "<JWT>", userName: "alice", password: "" };

export default function App() {
  const client = useContext(ZoomContext);
  const initialized = useRef(false);
  const [status, setStatus] = useState<"connecting" | "connected" | "closed">("connecting");
  const [mediaStream, setMediaStream] = useState<typeof Stream | null>(null);

  useEffect(() => {
    if (initialized.current) return;
    initialized.current = true;

    const join = async () => {
      await client.init("en-US", "Global", { patchJsMedia: true });
      try {
        await client.join(SESSION.topic, SESSION.token, SESSION.userName, SESSION.password);
        setMediaStream(client.getMediaStream());
        setStatus("connected");
      } catch (e) {
        console.error("join failed", e);
        setStatus("closed");
      }
    };
    join();

    return () => {
      if (client.getSessionInfo()?.isInMeeting) {
        ZoomVideo.destroyClient();
      }
    };
  }, [client]);

  const mediaContext = useMemo(() => ({ mediaStream }), [mediaStream]);

  if (status !== "connected" || !mediaStream) {
    return <div>Joining…</div>;
  }

  return (
    <ZoomMediaContext.Provider value={mediaContext}>
      <VideoSession />
    </ZoomMediaContext.Provider>
  );
}
```

## 4. Connection state

Handle every `ConnectionState` explicitly — failover/reconnect is normal and
must not look like a hard failure.

```tsx
import { ConnectionState, ReconnectReason } from "@zoom/videosdk";

useEffect(() => {
  const onConnectionChange = (payload: any) => {
    switch (payload.state) {
      case ConnectionState.Connected:
        setStatus("connected");
        break;
      case ConnectionState.Reconnecting:
        // Failover or moving between main session / subsession. Show a
        // non-blocking "reconnecting" UI, do NOT tear down the session.
        if (payload.reason === ReconnectReason.Failover) {
          // session dropped, SDK is recovering
        }
        setStatus("connecting");
        break;
      case ConnectionState.Closed:
        setStatus("closed");
        if (payload.reason === "ended by host") {
          // notify the user
        }
        break;
      case ConnectionState.Fail:
        setStatus("closed");
        // join/connection failed: payload.reason has the cause
        break;
    }
  };

  client.on("connection-change", onConnectionChange);
  return () => {
    client.off("connection-change", onConnectionChange);
  };
}, [client]);
```

## 5. Participant synchronization

Reuse one hook to keep the participant list in sync. It refreshes from
`getAllUser()` on every membership/state change and passes the changed subset so
callers can apply targeted updates (e.g. the current user during failover).

```typescript
// src/hooks/useParticipantsChange.ts
import { useCallback, useEffect, useRef } from "react";
import { useMount } from "./useMount";
import type { VideoClient, Participant } from "@zoom/videosdk";

export function useParticipantsChange(
  client: typeof VideoClient,
  fn: (participants: Participant[], updated?: Participant[]) => void
) {
  const fnRef = useRef(fn);
  fnRef.current = fn;

  const callback = useCallback(
    (updated?: Participant[]) => {
      fnRef.current(client.getAllUser(), updated);
    },
    [client]
  );

  useEffect(() => {
    client.on("user-added", callback);
    client.on("user-removed", callback);
    client.on("user-updated", callback);
    return () => {
      client.off("user-added", callback);
      client.off("user-removed", callback);
      client.off("user-updated", callback);
    };
  }, [client, callback]);

  useMount(() => callback());
}
```

Usage:

```tsx
const [participants, setParticipants] = useState<Participant[]>([]);

useParticipantsChange(client, (all) => {
  const self = client.getCurrentUserInfo();
  // Put video-on users first; keep self in a known position for layout.
  const others = all
    .filter((p) => p.userId !== self?.userId)
    .sort((a, b) => Number(b.bVideoOn) - Number(a.bVideoOn));
  setParticipants(others);
});
```

## 6. Rendering video (declarative `video-player`)

The official, idiomatic React pattern is to **declare `<video-player>` elements
inside a `<video-player-container>` in JSX**, capture each player via a `ref`,
and pass that element as the 3rd argument to `attachVideo`. Do not append
returned elements by hand.

Key rules:

- One `<video-player-container>` may hold many `<video-player>` children (gallery).
- Render a `<video-player>` for a user **only when `user.bVideoOn` is true**.
- `attachVideo(userId, quality, element)` renders into the passed element.
- `detachVideo(userId)` stops the stream and releases the element.
- Drive attach/detach from a `subscribers` list (visible users with video on),
  diffing against the previous list — never attach the same user twice.

```tsx
import { useContext, useEffect, useRef, useState } from "react";
import { VideoQuality } from "@zoom/videosdk";
import type { VideoPlayer, Participant } from "@zoom/videosdk";
import { ZoomContext } from "./context/zoom-context";
import { ZoomMediaContext } from "./context/media-context";
import { usePrevious } from "./hooks/usePrevious";
import { useParticipantsChange } from "./hooks/useParticipantsChange";

export default function VideoGallery() {
  const client = useContext(ZoomContext);
  const { mediaStream } = useContext(ZoomMediaContext);
  const playersRef = useRef<Record<string, VideoPlayer>>({});

  const [users, setUsers] = useState<Participant[]>(client.getAllUser());
  useParticipantsChange(client, () => setUsers(client.getAllUser()));

  // The set of userIds whose video should currently be rendered.
  const subscribers = users.filter((u) => u.bVideoOn).map((u) => u.userId);
  const prevSubscribers = usePrevious(subscribers) ?? [];

  const setPlayerRef = (userId: number, el: VideoPlayer | null) => {
    if (el) playersRef.current[userId] = el;
    else delete playersRef.current[userId];
  };

  useEffect(() => {
    if (!mediaStream) return;
    const added = subscribers.filter((id) => !prevSubscribers.includes(id));
    const removed = prevSubscribers.filter((id) => !subscribers.includes(id));

    removed.forEach((userId) => {
      mediaStream.detachVideo(userId);
      delete playersRef.current[userId];
    });
    added.forEach((userId) => {
      const el = playersRef.current[userId];
      if (el) mediaStream.attachVideo(userId, VideoQuality.Video_720P, el);
    });
  }, [mediaStream, subscribers, prevSubscribers]);

  return (
    <video-player-container class="video-grid">
      {users.map((user) => (
        <div className="video-cell" key={user.userId}>
          {user.bVideoOn && (
            <video-player
              class="video-player"
              ref={(el) => setPlayerRef(user.userId, el)}
            />
          )}
          {!user.bVideoOn && <div className="avatar">{user.displayName}</div>}
        </div>
      ))}
    </video-player-container>
  );
}
```

> **Self-view** uses the same `attachVideo` / `detachVideo` calls with the
> current user's `userId`. On some platforms (e.g. non-SAB Chromium, mobile)
> self-view must render to a `<video>` element; check
> `mediaStream.isRenderSelfViewWithVideoElement()`. With the `video-player`
> attach API the SDK handles this internally.

## 7. Layouts (1:1, speaker, gallery)

Layout is plain React state on top of the same render mechanism. Keep SDK attach
logic separate from layout/CSS so switching views never re-attaches video.

- **Gallery**: render every `participant` as a cell; size the grid with
  `gridTemplateColumns: repeat(cols, 1fr)`. Paginate when the participant count
  exceeds `mediaStream.getMaxRenderableVideos()` and only subscribe to the
  current page.
- **Speaker**: render the active speaker (or spotlighted user) large; render a
  thumbnail strip for the rest. Track the active speaker via the `active-speaker`
  event — **the payload is an array; read `payload[0].userId`** (not `oderId`).
- **1:1**: render the single remote participant large with self-view as a small
  draggable overlay.

```tsx
// Active speaker for speaker view
const [activeSpeakerId, setActiveSpeakerId] = useState(0);
useEffect(() => {
  const onActiveSpeaker = (payload: Array<{ userId: number }>) => {
    if (payload.length > 0) setActiveSpeakerId(payload[0].userId);
  };
  client.on("active-speaker", onActiveSpeaker);
  return () => client.off("active-speaker", onActiveSpeaker);
}, [client]);
```

For the full pagination + spotlight + grid-sizing implementation, see the
sample's `useAttachPagination`, `useSpotlightVideo`, `useGridLayout`, and
`useVideoGridStyle` hooks (`feature/video/hooks`).

## 8. Screen share

Two independent concerns:

1. **Sending** your own screen: choose `<video>` vs `<canvas>` with
   `isStartShareScreenWithVideoElement()`, then `startShareScreen(element)` /
   `stopShareScreen()`.
2. **Receiving** a peer's share: render it into a `<video-player>` (or
   `<video-player-container>`) with `attachShareView(userId, element)` /
   `detachShareView(userId, element)`.

Drive receive-side state from these events:

- `active-share-change` → `{ state: 'Active' | 'Inactive', userId }`: a peer
  started/stopped the primary share.
- `peer-share-state-change` → re-read `mediaStream.getShareUserList()` for the
  multi-sharer case.
- `share-content-dimension-change` → resize the share view to keep aspect ratio.
- `passively-stop-share` → your share was stopped (e.g. another user started).

```tsx
const [receiving, setReceiving] = useState(false);
const [activeShareId, setActiveShareId] = useState(0);
const shareRef = useRef<VideoPlayer | null>(null);

useEffect(() => {
  const onActiveShareChange = ({ state, userId }: any) => {
    setActiveShareId(userId);
    setReceiving(state === "Active");
  };
  client.on("active-share-change", onActiveShareChange);
  return () => client.off("active-share-change", onActiveShareChange);
}, [client]);

useEffect(() => {
  if (!mediaStream || !shareRef.current) return;
  if (receiving) mediaStream.attachShareView(activeShareId, shareRef.current);
  else mediaStream.detachShareView(activeShareId, shareRef.current);
}, [mediaStream, receiving, activeShareId]);
```

For **simultaneous share** (sharing your screen while also viewing a peer's
share) and **multi-share** (several share tiles at once), gate on the
`getMaxRenderableShareViews()` capability and track `getShareUserList()`; see the
sample's `useShare`, `useMultiShare`, and the `share-view/` components
(`single-share-view`, `multi-share-view`).

## 9. Annotation

Annotation runs on top of an active share. Capability is dynamic
(`mediaStream.canDoAnnotation()` depends on whether the user is the presenter or
a permitted viewer), so recompute it on share and privilege changes.

Toggle with `startAnnotation()` / `stopAnnotation()`. Stop annotation when:

- the active sharer changes,
- the share is paused (`ShareStatus.Paused`),
- annotation privilege is revoked.

Relevant events: `annotation-redo-status`, `annotation-undo-status`,
`annotation-viewer-draw-request` (a viewer must `startAnnotation()` on first
draw), `annotation-privilege-change`, and `share-content-dimension-change`. See
the sample's `useAnnotation` hook and `components/annotation/`.

## 10. Audio / video controls

Start audio from a user gesture. Guard every toggle against duplicate clicks and
keep a loading state until the Promise settles.

```tsx
const [audioOn, setAudioOn] = useState(false);
const [muted, setMuted] = useState(true);
const [busy, setBusy] = useState(false);

const toggleAudio = async () => {
  if (busy || !mediaStream) return;
  setBusy(true);
  try {
    if (!audioOn) {
      await mediaStream.startAudio(); // joins UNMUTED by default
      await mediaStream.muteAudio(); // mute explicitly for privacy
    } else if (muted) {
      await mediaStream.unmuteAudio();
    } else {
      await mediaStream.muteAudio();
    }
  } finally {
    setBusy(false);
  }
};
```

Keep local UI in sync with the authoritative events rather than only optimistic
state:

- `current-audio-change` → `{ action: 'join' | 'leave' | 'muted' | 'unmuted' }`.
- `video-active-change` / `peer-video-state-change` → drive video on/off and the
  attach/detach effect in §6.

## React-specific Pitfalls

- **Do not** call `ZoomVideo.createClient()` in a component/effect. Create it
  once outside the tree (or in a top-level module) and share via Context.
- **Do not** `appendChild` the value returned by `attachVideo`/`attachShareView`
  when you already declared a `<video-player>`; pass the element ref as the 3rd
  argument instead. The return value can also be an `ExecutedFailure`.
- **Only render `<video-player>` when `bVideoOn` is true**, and diff
  subscribers; otherwise you attach the same user repeatedly or leak players.
- **`active-speaker` payload is an array** — read `payload[0].userId`.
- **Don't tear down on `Reconnecting`.** Failover/subsession moves emit
  `ConnectionState.Reconnecting`; show a transient state and wait for
  `Connected`.
- **Clean up** every `client.on(...)` in the effect's cleanup, and
  `detachVideo` / `detachShareView` on unmount.
- The share-view container must be **separate** from the video container — do not
  attach video and share view into the same `<video-player-container>`.

## Reference Map (official sample)

When you need the full implementation, read these files from the official sample
repo: **<https://github.com/zoom/videosdk-web-sample>** (all paths are under
`src/`).

| Concern | File |
| --- | --- |
| Client context | [`context/zoom-context.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/context/zoom-context.ts) |
| Media (stream) context | [`context/media-context.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/context/media-context.ts) |
| Init/join + connection-change | [`App.tsx`](https://github.com/zoom/videosdk-web-sample/blob/master/src/App.tsx) |
| Participant sync | [`feature/video/hooks/useParticipantsChange.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/hooks/useParticipantsChange.ts) |
| Video render (declarative) | [`feature/video/video-attach.tsx`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/video-attach.tsx) |
| Gallery render (canvas, non-SAB) | [`feature/video/video.tsx`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/video.tsx), [`hooks/useGalleryLayout.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/hooks/useGalleryLayout.ts), [`hooks/useRenderVideo.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/hooks/useRenderVideo.ts) |
| Layout/pagination/spotlight | [`hooks/useGridLayout.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/hooks/useGridLayout.ts), [`useAttachPagination.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/hooks/useAttachPagination.ts), [`useSpotlightVideo.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/hooks/useSpotlightVideo.ts), [`useVideoGridStyle.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/hooks/useVideoGridStyle.ts) |
| Screen share | [`feature/video/hooks/useShare.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/hooks/useShare.ts), [`useMultiShare.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/hooks/useMultiShare.ts), [`components/share-view/`](https://github.com/zoom/videosdk-web-sample/tree/master/src/feature/video/components/share-view) |
| Annotation | [`feature/video/hooks/useAnnotation.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/hooks/useAnnotation.ts), [`components/annotation/`](https://github.com/zoom/videosdk-web-sample/tree/master/src/feature/video/components/annotation) |
| Active speaker / active video | [`feature/video/hooks/useAvtiveVideo.ts`](https://github.com/zoom/videosdk-web-sample/blob/master/src/feature/video/hooks/useAvtiveVideo.ts) |
