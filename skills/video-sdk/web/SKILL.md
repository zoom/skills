---
name: video-sdk-web
description: Build and debug browser-based Zoom Video SDK for Web integrations using @zoom/videosdk. Use for custom video sessions, joining or leaving sessions, JWT auth, audio/video and video-player rendering, screen sharing, chat and file transfer, command channel, recording, subsessions, transcription and translation, RTMS, live or broadcast streaming, raw data, whiteboard, PSTN/SIP, preview devices, browser support, SDK events, exact API/type lookup, or SDK errors. This is for custom Video SDK sessions, not Zoom Meeting SDK embedded meetings.
---

# Zoom Video SDK Web

This skill helps developers integrate Zoom Video SDK for Web into an existing
frontend application. Optimize for working code first, then add feature-specific
behavior and troubleshooting.

The primary build guidance and nested references are synchronized from the
official `zoom/videosdk-web` repository at SDK version `2.5.0`. Repo-local
concepts and examples complement that upstream material rather than override it.

Use repo-local official docs under `references/` as the primary source of truth.
Use `troubleshooting/` for symptom-driven debugging.

## Scope

Use this skill for:

- Browser integrations using `@zoom/videosdk`.
- Custom session UI: audio, video, screen share, chat, command channel,
  recording, subsessions, transcription/translation, PSTN/SIP, whiteboard,
  preview devices, and quality reporting.
- Debugging rejected Promises, SDK error codes, browser support, rendering
  issues, and event/lifecycle bugs.

Do not use this skill for:

- Joining or embedding regular Zoom Meetings or Webinars with Meeting SDK.
- Generic WebRTC advice that bypasses Video SDK APIs.
- Server-side Video SDK APIs except when generating the client JWT.

## Operating Principles

**Never invent SDK API surface.** Do not guess, recall, or "reconstruct" method
names, event strings, enum values, option fields, or payload shapes from memory.
The SDK surface changes across versions and plausible-looking APIs are easy to
hallucinate. Before writing any SDK call, event name, or enum, confirm it exists
in the bundled type definitions (`node_modules/@zoom/videosdk/dist/types/`). If a
symbol cannot be verified there, say so explicitly and check the types or ask —
do not emit a plausible guess. This rule overrides convenience and speed.

1. Start with the shortest runnable path for the developer's current project.
2. Detect the current framework from the repo before giving framework-specific
   code.
3. Read the relevant docs before making non-trivial claims or changes.
4. Prefer official `references/` content over memory.
5. Do not provide large API inventories in the answer unless the user asks.
6. Treat most SDK operations as async Promise-returning operations; prevent
   duplicate in-flight calls in UI.
7. Use `error.errorCode` for actionable troubleshooting.

Useful type entry points (paths relative to your project root, after
`npm install @zoom/videosdk`):

- Session/client: `node_modules/@zoom/videosdk/dist/types/videoclient.d.ts`
- Events: `node_modules/@zoom/videosdk/dist/types/event-callback.d.ts`
- Audio / video / screen share: `node_modules/@zoom/videosdk/dist/types/media.d.ts`
- Chat: `node_modules/@zoom/videosdk/dist/types/chat.d.ts`
- Command channel: `node_modules/@zoom/videosdk/dist/types/command.d.ts`
- Recording: `node_modules/@zoom/videosdk/dist/types/recording.d.ts`
- Transcription: `node_modules/@zoom/videosdk/dist/types/live-transcription.d.ts`
- Error codes: `node_modules/@zoom/videosdk/dist/types/exception-code.d.ts`

When the package is not installed, or when an AI-readable declaration inventory
is more efficient, read `references/type-definitions-json.md` and then only the
matching file under `references/types-json/`. These generated files describe
`@zoom/videosdk` `2.5.0`; prefer the installed package's `.d.ts` files whenever
the application's package version differs.

## First-Step Triage

Before implementing or answering in detail, identify:

- Package mode: npm package (`@zoom/videosdk`) or CDN.
- Framework: vanilla JavaScript/TypeScript, React, Vue, Svelte, Angular, or other.
- Target task: shortest runnable session, specific feature, layout, or debugging.
- Runtime constraints: desktop/mobile browser, HTTPS/dev server, COOP/COEP/SAB
  requirements, user gesture requirements.

Suggested repo checks:

```bash
rg -n "@zoom/videosdk|ZoomVideo|WebVideoSDK|video-player-container|attachVideo" .
rg -n "\"react\"|\"vue\"|\"svelte\"|\"@angular/core\"|vite|next|nuxt|angular" package.json . -g 'package.json' -g 'vite.config.*' -g 'angular.json'
```

## Read Routing

Always read only what is needed for the user's task.

### Shortest Runnable Session

Read these first:

- `references/get_started.md`
- `references/auth.md`
- `references/sessions.md`
- `references/video/video.md`
- `references/audio/audio.md`
- `references/screen-sharing/share.md`
- `references/handle_events.md`

Use this route when the user wants to join a session, start audio, start video,
see/hear remote users, start screen share, or receive screen share.

### Framework Integration

Read `references/frameworks.md` first, then read framework-specific guidance only
when the current project uses that framework:

- React: `references/framework-integration/react.md`
- Next.js: `references/framework-integration/nextjs.md`
- Vue: `references/framework-integration/vue.md`
- Nuxt: `references/framework-integration/nuxtjs.md`
- Svelte: `references/framework-integration/svelte.md`
- Angular: `references/framework-integration/angular.md`

Keep framework code idiomatic to the existing project. Do not force a new state
management library or UI kit.

### Events and State Synchronization

Read:

- `references/handle_events.md`

Use for participant list sync, mid-session join, reconnecting users,
`isInFailover`, connection state, audio/video/share events, and stale UI.

### Troubleshooting

Read:

- `troubleshooting/common-issues.md`
- `references/error-codes.md`
- `references/features/quality.md` when preparing logs or reporting issues to Zoom.
- `references/browser-support.md` when behavior is browser/platform-specific.

Use rejected Promise `error.errorCode` first, then match known common issues.

### Feature-Specific Routing

| User asks about | Read |
| --- | --- |
| Audio, devices, high bitrate, original sound, noise suppression | `references/audio/audio.md`, `references/audio/audio-advanced.md`, `references/audio/audio-sound-options.md`, `references/audio/audio-best-practices.md` |
| Video rendering, gallery view, speaker view, 1:1 layout, active speaker | `references/video/video.md`, `references/video/video-best-practices.md` |
| HD video, 720p/1080p, camera support | `references/video/video-hd.md`, `references/browser-support.md` |
| Virtual background, PTZ, PiP, second camera | `references/video/video-advanced.md`, `references/video/video-camera-controls.md`, `references/video/video-picture-in-picture.md` |
| Screen sharing, receive share, multiple shares, share layout | `references/screen-sharing/share.md` |
| Annotation or share audio/system audio | `references/screen-sharing/share-annotation.md`, `references/screen-sharing/share-browser-options.md` |
| Command channel or custom in-session control messages | `references/features/command-channel.md` |
| Chat | `references/chat/chat.md`, `references/chat/chat-send-files.md` |
| Recording | `references/features/recording.md` |
| Subsessions, breakout rooms, waiting-room-like flows | `references/features/subsessions.md`, `troubleshooting/common-issues.md` |
| Live transcription, translation, captions | `references/features/transcription-translation.md` |
| PSTN/SIP phone call | `references/features/pstn.md`, `references/features/sip.md` |
| Whiteboard | `references/features/whiteboard.md` |
| Preview microphone/camera before session | `references/features/preview.md` |
| Live stream / RTMP | `references/features/live-stream.md`, `references/features/incoming-live-stream.md` |
| Broadcast streaming | `references/features/broadcast.md` |
| Raw audio, video, or share data | `references/raw-data/raw-data.md`, `references/raw-data/raw-data-audio.md`, `references/raw-data/raw-data-video.md`, `references/raw-data/raw-data-share.md` |
| Browser support, SAB, CSP, COOP/COEP | `references/browser-support.md` |

## Shortest Runnable Implementation

Use this as the default target when the user asks for a working integration.
Adapt it to the detected framework.

### Install SDK

NPM:

```bash
npm install @zoom/videosdk
```

CDN:

```html
<script src="https://source.zoom.us/videosdk/zoom-video-#.#.#.min.js"></script>
```

Prefer npm in modern frontend projects. Use CDN only when the existing app is
script-based or explicitly asks for CDN. With CDN, access the SDK from
`window.WebVideoSDK.default`.

### Generate JWT on Server

Never expose the Video SDK secret in client code.

Required JWT claims:

```javascript
{
  app_key: process.env.ZOOM_SDK_KEY,
  tpc: sessionName,
  role_type: 0, // 0 participant, 1 host/co-host
  version: 1,
  iat,
  exp,
}
```

Important claims:

- `app_key`: Video SDK key.
- `role_type`: `1` host/co-host, `0` participant. Must be a number.
- `tpc`: session name, max 200 chars, must match `client.join(topic, ...)`.
- `user_key`: stable user/customer identifier for auditing.
- `session_key`: stable session identifier; all attendees must use the same
  value if host sets it.
- `telemetry_tracking_id`: useful when reporting Web SDK issues to Zoom.
- `video_webrtc_mode` / `audio_webrtc_mode`: JWT-level WebRTC mode hints, not
  `client.init()` options.

Reference: `references/auth.md`.

### Minimal HTML/CSS

`video-player` elements returned by `attachVideo()` must be appended inside a
`video-player-container`. Give the container and players dimensions.

The SDK owns the rendering surface of `video-player` and `video-player-container`.
Avoid setting an opaque `background` / `background-color` / `background-image`
directly on `video-player`, `video-player-container`, or an element nested between
them while video is showing — a full-area opaque background on these elements can
paint over the rendered video and make it disappear, depending on the runtime.
Overlaying UI on top of the video (name tags, mic/status badges, controls) is
fine: add it as a separately positioned child element, not as a background on the
player/container. For a placeholder/letterbox color shown before video attaches,
prefer an outer wrapper behind the container, and remove or hide it once video is
attached.

```html
<button id="join">Join</button>
<button id="start-audio">Start audio</button>
<button id="start-video">Start video</button>
<button id="start-share">Start share</button>

<video-player-container class="video-grid"></video-player-container>

<video id="local-share-video" width="1920" height="1080"></video>
<canvas id="local-share-canvas" width="1920" height="1080"></canvas>
<video-player-container class="share-container"></video-player-container>
```

```css
video-player-container.video-grid,
video-player-container.share-container {
  width: 100%;
  min-height: 360px;
  display: flex !important;
  flex-wrap: wrap;
  align-content: baseline;
  gap: 8px;
}

video-player {
  width: 100%;
  height: auto;
  flex: 0 0 50%;
  aspect-ratio: 16 / 9;
}

#local-share-video,
#local-share-canvas {
  width: 100%;
  height: auto;
}
```

### Minimal Client Flow

```javascript
import ZoomVideo, { VideoQuality } from "@zoom/videosdk";

const client = ZoomVideo.createClient();
let stream;
let activeShareUserId = null;
let startingAudio = false;
let startingVideo = false;
let startingShare = false;

async function joinSession({ topic, token, userName, password }) {
  const support = ZoomVideo.checkSystemRequirements();
  if (!support.audio || !support.video) {
    throw new Error("Browser does not support required Video SDK features.");
  }

  await client.init("en-US", "Global", { patchJsMedia: true });
  await client.join(topic, token, userName, password);

  stream = client.getMediaStream();
  bindEvents();
  await renderExistingVideos();
  await renderExistingShare();
}

async function startAudio() {
  if (startingAudio) return;
  startingAudio = true;
  try {
    await stream.startAudio();
  } finally {
    startingAudio = false;
  }
}

async function startVideo() {
  if (startingVideo) return;
  startingVideo = true;
  try {
    await stream.startVideo();
    const userId = client.getCurrentUserInfo().userId;
    await attachUserVideo(userId);
  } finally {
    startingVideo = false;
  }
}

async function startShare() {
  if (startingShare) return;
  startingShare = true;
  try {
    if (stream.isStartShareScreenWithVideoElement()) {
      await stream.startShareScreen(document.querySelector("#local-share-video"));
    } else {
      await stream.startShareScreen(document.querySelector("#local-share-canvas"));
    }
  } finally {
    startingShare = false;
  }
}

function bindEvents() {
  client.on("peer-video-state-change", async ({ action, userId }) => {
    if (action === "Start") {
      await attachUserVideo(userId);
    } else if (action === "Stop") {
      removeDetachedElements(await stream.detachVideo(userId));
    }
  });

  client.on("active-share-change", async ({ state, userId }) => {
    if (state === "Active") {
      await attachShare(userId);
    } else if (state === "Inactive") {
      removeDetachedElements(await stream.detachShareView(userId));
      activeShareUserId = null;
    }
  });

  client.on("user-updated", (users) => {
    users.forEach((user) => {
      if (user.isInFailover) {
        // Keep the user visible, but show reconnecting/unstable status.
      }
    });
  });

  // Always handle connection-change: it is the source of truth for session
  // state. Without it the UI can keep showing "in session" after the user has
  // actually disconnected, which desyncs your app from reality.
  client.on("connection-change", (payload) => {
    if (payload.state === "Connected") {
      // Session is live; clear any reconnecting/closed UI.
    } else if (payload.state === "Reconnecting") {
      // Lost connection, SDK is retrying. Show a reconnecting banner; do not
      // tear down session UI yet.
    } else if (payload.state === "Closed") {
      // Session ended (host ended it, or the user was removed). Tear down
      // session UI and route the user out. payload.reason explains why.
    } else if (payload.state === "Fail") {
      // Join/reconnect failed permanently. Surface an error and leave.
      console.error("Connection failed", payload.errorCode, payload.reason);
    }
  });

  // Always handle active-media-failed: media can fail after starting (permission
  // reset, device taken by another app, stream interrupted). Turn payload.code
  // into a user-facing recovery hint instead of failing silently.
  client.on("active-media-failed", (payload) => {
    // payload.code (ActiveMediaFailedCode) + payload.message. Map the code to a
    // suggested action and show it to the user (grant permission, click the
    // page to resume, refresh, etc.). See references/handle_events.md and
    // references/error-codes.md for the full code-to-action table.
    console.error("active-media-failed", payload.code, payload.message);
  });
}

async function renderExistingVideos() {
  for (const user of client.getAllUser()) {
    if (user.bVideoOn) {
      await attachUserVideo(user.userId);
    }
  }
}

async function attachUserVideo(userId) {
  const video = await stream.attachVideo(userId, VideoQuality.Video_360P);
  document.querySelector("video-player-container.video-grid").appendChild(video);
}

async function renderExistingShare() {
  const sharingUser = client.getAllUser().find((user) => user.sharerOn);
  if (sharingUser) {
    await attachShare(sharingUser.userId);
  }
}

async function attachShare(userId) {
  if (activeShareUserId && activeShareUserId !== userId) {
    removeDetachedElements(await stream.detachShareView(activeShareUserId));
  }

  const shareView = await stream.attachShareView(userId);
  document.querySelector("video-player-container.share-container").appendChild(shareView);
  activeShareUserId = userId;
}

function removeDetachedElements(detached) {
  if (Array.isArray(detached)) {
    detached.forEach((element) => element.remove());
  } else if (detached) {
    detached.remove();
  }
}
```

## Event Model Guidance

Use SDK events as state change signals, then reconcile with SDK getters when
needed.

Key events:

- `connection-change`: join failure, reconnecting, closed, connected.
- `user-added`, `user-removed`, `user-updated`: participant list and failover.
- `current-audio-change`: local or remote audio state changes.
- `peer-video-state-change`: render/detach remote video.
- `active-share-change`: current active share view.
- `peer-share-state-change`: multiple-share flows.
- `device-change` and `device-permission-change`: device picker and permission UI.
- `active-media-failed`: media failure requiring user intervention.

Always bind these two — they are not optional, even for a minimal integration:

- `connection-change` is the source of truth for session lifecycle. Without it
  the app can keep showing "in session" after the user has actually
  disconnected, desyncing the UI from reality. Handle every state (`Connected`,
  `Reconnecting`, `Closed`, `Fail`), not just failure: show a reconnecting state
  on `Reconnecting`, and route the user out on `Closed`/`Fail`.
- `active-media-failed` fires when media fails *after* it started (permission
  reset, device taken by another app, interrupted stream, WebGL/WASM issues).
  Map `payload.code` to a concrete recovery hint for the user (grant permission,
  click the page to resume, refresh) instead of failing silently. The full
  code-to-action table is in `references/handle_events.md`.

Important: a disconnected user may remain in the session briefly because of
server heartbeat/failover handling. Use `user.isInFailover` to show unstable or
reconnecting UI instead of assuming the user has cleanly left.

## Feature Guidance

### Command Channel

Suggest command channel for low-frequency custom in-session controls such as
reactions, layout hints, control messages, and app-specific state updates.

Constraints:

- Send strings only; JSON must be `JSON.stringify(...)`.
- Maximum message size is 512 characters.
- Rate limit is 2 commands per second per session by default.
- Not designed for high-frequency reliable N-to-N broadcast. Use a dedicated
  signaling service for that.

### Audio

Start audio from a user gesture when possible. Use loading state to prevent
duplicate `startAudio()` calls. Read advanced docs for high bitrate, original
sound, stereo, background noise suppression, and device handling.

### Video Layouts

Use `video-player-container` as the SDK render host, and build app layout around
returned `video-player` elements. For gallery, speaker, 1:1, and share-combined
layouts, keep SDK attachment logic separate from CSS/layout state.

Keep the player and container transparent: put name tags, badges, and controls in
positioned overlay children, never as a full-area opaque background on
`video-player` / `video-player-container`, which can hide the rendered video.

Reuse the `video-player-container`; do not destroy and recreate it. The container
holds the shared rendering surface for all videos under it, and the browser caps
how many such surfaces can exist — repeatedly tearing the container down and
recreating it churns that surface and can exhaust the limit, after which video
stops rendering entirely. Mount one `video-player-container` per render host and
keep it for the session lifetime: do not unmount/remount it on route changes, tab
switches, or layout toggles. In React/Vue, do not place it behind conditional
rendering that destroys it — keep it mounted and toggle visibility with CSS
(`display: none` / `visibility: hidden`) instead. Individual `video-player`
elements may be freely created and removed via `attachVideo()` / `detachVideo()`;
only the container must stay stable.

### Screen Share

Use `stream.isStartShareScreenWithVideoElement()` to choose video vs canvas when
starting local share. Use `attachShareView()` / `detachShareView()` to receive
remote shares. For simultaneous shares, read `references/screen-sharing/share.md` before
implementing.

### Preview

Use preview APIs for pre-session camera and microphone testing. Do not join a
session only to test devices.

## Troubleshooting Workflow

1. Capture the rejected Promise object and inspect `error.errorCode`,
   `error.type`, and `error.reason`.
2. Match `error.errorCode` against `references/error-codes.md`.
3. Check `troubleshooting/common-issues.md` for known symptoms and fixes.
4. Verify the relevant official doc in `references/`.
5. For browser-specific behavior, check `references/browser-support.md`.
6. For issues requiring Zoom investigation, follow `references/features/quality.md` and
   include telemetry tracking ID, SDK version, browser, OS, session details, and
   reproducible steps.

Common implementation checks:

- JWT `tpc` matches `client.join(topic, ...)`.
- JWT `exp` is between 30 minutes and 48 hours after `iat`.
- Video SDK secret is never exposed in frontend code.
- `client.init()` completed before `client.join()`.
- Media operations are called after join and guarded against duplicate clicks.
- `video-player` has dimensions through CSS and is inside `video-player-container`.
- `video-player` / `video-player-container` have no full-area opaque background;
  overlays are positioned children, not backgrounds (a covering background hides
  video).
- `video-player-container` is mounted once and reused for the session, not
  destroyed/recreated on route, tab, or layout changes (recreating it can exhaust
  the browser's rendering-surface limit and stop video from rendering).
- `connection-change` is bound and handles `Reconnecting`/`Closed`/`Fail`, not
  just the happy path (otherwise the UI desyncs when the session drops).
- `active-media-failed` is bound and surfaces a user-facing recovery hint from
  `payload.code`.
- Remote videos are rendered from both `peer-video-state-change` and a post-join
  `client.getAllUser()` reconciliation pass.
- Screen share receive uses `attachShareView()` / `detachShareView()`.
- Browser support is checked before exposing unsupported features.

## Best Practices

- Generate JWT on a secure backend.
- Keep session lifecycle, media controls, event listeners, and DOM rendering
  separated in code.
- Bind core events before or immediately after join, then reconcile with getters.
- Use loading/in-flight flags for `join`, `startAudio`, `startVideo`,
  `startShareScreen`, recording start/stop, and other async operations.
- Clean up event listeners and detached media elements on leave/unmount.
- Avoid hard-coding HD, SharedArrayBuffer, or WebRTC assumptions; check browser
  support and feature docs.
- Prefer concise, project-specific implementation guidance over dumping all SDK
  APIs.

## Reference Index

For the full list of reference files and when to read each, use the routing
tables above: the "Read Routing" section for core/session/event/troubleshooting
docs, the "Feature-Specific Routing" table for per-feature docs, and the
"Framework Integration" section for framework docs.

For exact API names, signatures, enums, event payloads, and declaration source
locations, use `references/type-definitions-json.md`.

## Repo-Local Appendices

Read these only when they add detail not covered by the official build path:

- Operations and maintenance: `RUNBOOK.md`, `MAINTENANCE.md`
- Architecture: `concepts/sdk-architecture-pattern.md`, `concepts/singleton-hierarchy.md`
- Session implementation: `examples/session-join-pattern.md`, `examples/event-handling.md`
- Media implementation: `examples/audio.md`, `examples/video-rendering.md`, `examples/screen-share.md`
- Feature examples: `examples/chat.md`, `examples/command-channel.md`, `examples/recording.md`, `examples/transcription.md`
- Framework examples: `examples/react-hooks.md`, `examples/framework-integrations.md`
- Consolidated legacy references: `references/web.md`, `references/web-reference.md`, `references/audio.md`, `references/video.md`, `references/screen-share.md`, `references/features.md`, `references/advanced.md`, `references/events-reference.md`, `references/common-issues.md`
- Extended framework references: `references/react.md`, `references/vue.md`, `references/angular.md`, `references/svelte.md`
- Reviewed 2.4.5 sample map: `references/sample-app-2.4.5.md`
