---
name: video-sdk/web
description: Expert guidance for building browser-based video sessions with the Zoom Video SDK for Web (@zoom/videosdk v2.4.0) in React, Vue, Angular, Svelte, or vanilla TypeScript. Use this skill whenever the user is implementing or debugging any in-browser real-time communication feature — joining/leaving a session, capturing or rendering audio/video, gallery or active-speaker views, virtual backgrounds, screen sharing with annotation, in-session chat or command channel, recording, subsessions, live streaming, PSTN/SIP dial-out, PTZ cameras, quality stats, WebAssembly/SharedArrayBuffer setup, CSP/COOP/COEP headers, JWT session tokens, or resolving SDK error codes. Trigger even when the user doesn't explicitly say "Zoom" — signals include `@zoom/videosdk`, `ZoomVideo.createClient`, `client.getMediaStream`, `stream.startVideo`, `attachVideo`, "video conferencing", "video call app", "video SDK", "render remote video", or debugging black/green video tiles, audio that won't start, or `OperationBlockedByBrowserPolicy` errors. Prefer this skill over generic WebRTC advice whenever `@zoom/videosdk` is in play.
triggers:
  - "video sdk web"
  - "custom video web"
  - "attachvideo"
  - "web videosdk"
  - "zoom videosdk web"
  - "@zoom/videosdk"
  - "video web session"
  - "peer-video-state-change"
  - "video rendering"
---

# Zoom Video SDK Web (v2.4.0)

Expert guidance for building video sessions with Zoom Video SDK for Web.

This skill is for **custom video sessions**, not embedded Zoom meetings.
If the user wants a custom UI for a real Zoom meeting, route to [../../meeting-sdk/web/component-view/SKILL.md](../../meeting-sdk/web/component-view/SKILL.md).
Use [../../probe-sdk/SKILL.md](../../probe-sdk/SKILL.md) as an optional browser/device/network readiness gate before `client.join(...)`.

## Quick Start

### Installation

```bash
bun install @zoom/videosdk --save
# or 
npm install @zoom/videosdk --save
```

### Basic Session Setup

```typescript
import ZoomVideo from "@zoom/videosdk";

// 1. Create client
const client = ZoomVideo.createClient();

// 2. Initialize SDK
await client.init("en-US", "Global", { patchJsMedia: true });

// 3. Join session (requires JWT token from your server)
await client.join(sessionName, jwtToken, userName, sessionPassword);

// 4. Get media stream for audio/video control
const stream = client.getMediaStream();
```

## Core API Reference

### ZoomVideo (Static Methods)

| Method                             | Description                                              |
| ---------------------------------- | -------------------------------------------------------- |
| `createClient()`                   | Create VideoClient instance (singleton)                  |
| `checkSystemRequirements()`        | Check browser compatibility → `{ audio, video, screen }` |
| `checkFeatureRequirements()`       | Get supported/unsupported features list                  |
| `getDevices(skipPermissionCheck?)` | Enumerate media devices                                  |
| `createLocalAudioTrack(deviceId?)` | Create local audio track for preview                     |
| `createLocalVideoTrack(deviceId?)` | Create local video track for preview                     |
| `destroyClient()`                  | Destroy client instance                                  |
| `preloadDependentAssets(path?)`    | Preload WebAssembly/Worker assets                        |

### VideoClient Methods

#### Session Management

```typescript
// Initialize before joining
await client.init(language, dependentAssets, options?);

// Join session
await client.join(topic, token, userName, password?, idleTimeoutMins?);

// Leave or end session
await client.leave(end?); // end=true ends for all (host only)
```

#### User Management

```typescript
client.getCurrentUserInfo(): Participant;
client.getAllUser(): Participant[];
client.getUser(userId): Participant | undefined;
client.getSessionHost(): Participant | undefined;

// Host/Manager actions
client.makeHost(userId);      // Transfer host
client.makeManager(userId);   // Promote to manager
client.revokeManager(userId); // Revoke manager
client.removeUser(userId);    // Remove participant
client.changeName(name, userId?);
```

#### Feature Clients

```typescript
client.getMediaStream(); // Audio/Video/Screen share
client.getChatClient(); // In-session chat
client.getCommandClient(); // Custom signaling
client.getRecordingClient(); // Cloud recording
client.getSubsessionClient(); // Breakout rooms
client.getLiveTranscriptionClient(); // Captions
client.getLiveStreamClient(); // RTMP streaming
client.getWhiteboardClient(); // Whiteboard
```

### MediaStream (Audio/Video Control)

#### Audio

```typescript
const stream = client.getMediaStream();

// Start audio (requires user gesture)
await stream.startAudio({
  mute?: boolean,
  speakerOnly?: boolean,
  backgroundNoiseSuppression?: boolean,
  microphoneId?: string,
  speakerId?: string,
});

// Control
await stream.muteAudio();
await stream.unmuteAudio();
stream.stopAudio();

// Device management
stream.getMicList(): MediaDevice[];
stream.getSpeakerList(): MediaDevice[];
await stream.switchMicrophone(deviceId);
await stream.switchSpeaker(deviceId);
```

#### Video

```typescript
// Start video (simple call, no options needed)
await stream.startVideo();

// Or with options
await stream.startVideo({
  cameraId?: string,
  hd?: boolean,           // 720p
  fullHd?: boolean,       // 1080p
  mirrored?: boolean,
  virtualBackground?: { imageUrl: string | 'blur' | undefined, cropped?: boolean },
});

// CRITICAL: Attach video to DOM
// 1. Container MUST be a <video-player-container> custom element
// 2. attachVideo returns an element - append it to the container
// 3. Do NOT pass container as third parameter
const videoElement = await stream.attachVideo(userId, VideoQuality.Video_720P);
container.appendChild(videoElement);

// Stop video
await stream.stopVideo();

// Detach video (cleanup)
stream.detachVideo(userId);

// Device management
stream.getCameraList(): MediaDevice[];
await stream.switchCamera(deviceId);
```

#### Screen Share

```typescript
// Start sharing
await stream.startShareScreen({
  broadcastToSubsession: boolean,
  optimizedForSharedVideo: boolean,
  secondaryCameraId: string, // Share secondary camera
});

// Stop sharing
await stream.stopShareScreen();

// View others' share
await stream.startShareView(canvas, userId);
stream.stopShareView();
```

### Events

```typescript
// Connection
client.on("connection-change", (payload) => {
  // payload.state: 'Connected' | 'Reconnecting' | 'Closed' | 'Fail'
});

// Users
client.on("user-added", (participants: Participant[]) => {});
client.on("user-updated", (participants: Participant[]) => {});
client.on("user-removed", (participants: Participant[]) => {});

// Audio
client.on("current-audio-change", (payload) => {
  // payload.action: 'join' | 'leave' | 'muted' | 'unmuted'
});
client.on("active-speaker", (payload) => {
  // payload.activeSpeaker: { oderId: number, oderId?: number }[]
});

// Video
client.on("video-active-change", (payload) => {
  // payload.userId, payload.state: 'Active' | 'Inactive'
});
client.on("peer-video-state-change", (payload) => {
  // payload.userId, payload.action: 'Start' | 'Stop'
});

// Screen Share
client.on("active-share-change", (payload) => {
  // payload.userId, payload.state: 'Active' | 'Inactive'
});
client.on("peer-share-state-change", (payload) => {
  // payload.userId, payload.action: 'Start' | 'Stop'
});

// Chat
client.on("chat-on-message", (payload) => {
  // payload.message, payload.sender, payload.timestamp
});

// Device
client.on("device-change", () => {
  // Re-enumerate devices
});
```

## Framework-Specific Implementation Guides

For complete implementation examples with full project setup, hooks/composables/services, and components:

- **[references/react.md](references/react.md)** - React 19 + Vite 7 + TypeScript + shadcn/ui implementation
- **[references/vue.md](references/vue.md)** - Vue 3 + Vite 7 + TypeScript + shadcn-vue implementation
- **[references/angular.md](references/angular.md)** - Angular 21 + Standalone Components + Signals implementation
- **[references/svelte.md](references/svelte.md)** - Svelte 5 + Runes + Vite 7 + TypeScript implementation

### Quick Setup Requirements

**All frameworks MUST configure:**

1. **COOP/COEP Headers** (for SharedArrayBuffer support):

```typescript
// vite.config.ts
server: {
  headers: {
    'Cross-Origin-Opener-Policy': 'same-origin',
    'Cross-Origin-Embedder-Policy': 'require-corp',
  },
}
```

2. **TypeScript Custom Elements** (for video rendering):

```typescript
// src/types/zoom-elements.d.ts
declare namespace JSX {
  interface IntrinsicElements {
    "video-player-container": React.DetailedHTMLProps<
      React.HTMLAttributes<HTMLElement>,
      HTMLElement
    >;
  }
}
```

### Core Implementation Pattern

All implementations should follow this pattern:

1. **Client Management** - Singleton client with init/join/leave lifecycle
2. **Media Stream** - Audio/video/screen share controls with state sync
3. **Participants** - User list with video state change listeners
4. **Video Rendering** - Use `video-player-container` + `attachVideo()`
5. **Event Handling** - Connection, audio, video, chat events
6. **Error Handling** - Map SDK error codes to user messages

## JWT Token Requirements

Generate JWT on your server using HMAC SHA256. Never expose SDK secret in client code.

### Required Claims

```javascript
{
  app_key: 'YOUR_SDK_KEY',    // Video SDK key
  tpc: 'session-name',        // Session name (max 200 chars), must match join() topic
  role_type: 1,               // 1 = host/co-host, 0 = participant
  version: 1,                 // Always set to 1
  iat: Math.floor(Date.now() / 1000) - 30,  // Issued at (subtract 30s for clock skew)
  exp: iat + 7200,            // Expiration: min 1800s, max 48 hours after iat
}
```

### Optional Claims

| Claim                               | Description                                                |
| ----------------------------------- | ---------------------------------------------------------- |
| `user_key`                          | Unique user identifier (max 36 chars)                      |
| `session_key`                       | Session identifier for cloud recording (max 36 chars)      |
| `geo_regions`                       | Data center regions: `AU,BR,CA,DE,HK,IN,JP,CN,MX,NL,SG,US` |
| `cloud_recording_option`            | `0` = combined video, `1` = separate files per user        |
| `cloud_recording_election`          | `1` to record user's self-view                             |
| `cloud_recording_transcript_option` | `0` = none, `1` = transcript, `2` = transcript + summary   |
| `video_webrtc_mode`                 | `0` = disable, `1` = enable WebRTC video                   |
| `audio_webrtc_mode`                 | `1` = enable WebRTC audio (Web only)                       |
| `telemetry_tracking_id`             | For Web SDK telemetry tracking                             |

### Node.js(jsrsasign) Quick Start JWT Generator Service:

```bash
git clone https://github.com/zoom/videosdk-auth-endpoint-sample.git

cd videosdk-auth-endpoint-sample

bun install
# or 
npm install
# Rename .env.example to .env, edit the file contents to include your Zoom Video SDK key and secret, save the file contents, and close the file
mv .env.example .env
# Start the server
# server code https://raw.githubusercontent.com/zoom/videosdk-auth-endpoint-sample/refs/heads/master/src/index.js
bun run start
# or
npm run start
```

jsrsasign Example Code:

```javascript
import { KJUR } from "jsrsasign";

const iat = Math.floor(Date.now() / 1000) - 30;
const exp = iat + 60 * 60 * 2; // 2 hours

const payload = {
  app_key: process.env.ZOOM_SDK_KEY,
  tpc: "session-name",
  role_type: 1,
  version: 1,
  video_webrtc_mode: 1,
  audio_webrtc_mode: 1,
  iat,
  exp,
};

const token = KJUR.jws.JWS.sign(
  "HS256",
  JSON.stringify({ alg: "HS256", typ: "JWT" }),
  JSON.stringify(payload),
  process.env.ZOOM_SDK_SECRET
);
```

## Init Options

```typescript
interface InitOptions {
  webEndpoint?: string; // Custom endpoint
  enforceMultipleVideos?: boolean | { disableRenderLimits?: boolean };
  enforceVirtualBackground?: boolean;
  stayAwake?: boolean; // Prevent screen dimming
  leaveOnPageUnload?: boolean; // Quick leave on page close
  patchJsMedia?: boolean; // Apply latest media fixes (recommended: true)
  alternativeNameForVideoPlayer?: string;
}
```

## Video Quality Enum

```typescript
enum VideoQuality {
  Video_90P = 0,
  Video_180P = 1,
  Video_360P = 2,
  Video_720P = 3,
  Video_1080P = 4,
}
```

## Best Practices

1. **Always check browser support** before initializing
2. **Generate JWT on server** - never expose SDK secret in client
3. **Handle connection events** for robust session management
4. **Request user gesture** before starting audio (browser requirement)
5. **Use `patchJsMedia: true`** for latest WebAssembly fixes
6. **Set `leaveOnPageUnload: true`** for clean disconnection
7. **Implement error handling** for all async operations
8. **Clean up event listeners** on session end

## Common Issues

| Issue                               | Solution                                                                      |
| ----------------------------------- | ----------------------------------------------------------------------------- |
| Audio doesn't start                 | Requires user gesture (click/tap)                                             |
| Video not rendering                 | Use `video-player-container` custom element and append `attachVideo()` result |
| JWT invalid                         | Verify `tpc` matches session name                                             |
| SharedArrayBuffer error             | **MUST** add COOP/COEP headers in vite.config.ts                              |
| `OPERATION_TIMEOUT` on startVideo   | Check camera permissions and ensure COOP/COEP headers are set                 |
| `INVALID_PARAMETERS` on attachVideo | Don't pass container as 3rd param - append the returned element instead       |
| Self-video not showing              | Pass `currentUserVideoOn` prop to VideoGrid (participant.bVideoOn may lag)    |
| Participants not updating           | Listen for `video-active-change` and `peer-video-state-change` events         |
| "Waiting for participants" stuck    | Pass `isJoined` to `useParticipants` hook and listen for `connection-change`  |

## Detailed Guides

### Core Concepts (read these first — they apply to every feature)

- [concepts/sdk-architecture-pattern.md](concepts/sdk-architecture-pattern.md) - The universal 5-step pattern (`createClient → init → join → getMediaStream → use`) that every feature follows
- [concepts/singleton-hierarchy.md](concepts/singleton-hierarchy.md) - Service-locator navigation tree from `ZoomVideo` root to every sub-client (media, chat, recording, subsession, command, etc.)

### Feature References

- [references/sessions.md](references/sessions.md) - Session lifecycle, user roles, JWT token, connection events
- [references/audio.md](references/audio.md) - Audio controls, devices, muting, dial-out, noise suppression
- [references/video.md](references/video.md) - Video capture, rendering, virtual background, PTZ cameras
- [references/screen-share.md](references/screen-share.md) - Screen sharing, annotation, tab audio, share privileges
- [references/features.md](references/features.md) - Preview, recording, subsessions, live stream, command channel, transcription, PSTN, SIP, quality stats
- [references/advanced.md](references/advanced.md) - Chat, custom processors, whiteboard
- [references/browser-support.md](references/browser-support.md) - Browser compatibility, SharedArrayBuffer, CSP headers
- [references/error-codes.md](references/error-codes.md) - Complete error code reference
- [troubleshooting/common-issues.md](troubleshooting/common-issues.md) - Troubleshooting guide: symptoms → causes → fixes for the most common Video SDK bugs

## Type Definitions Location

All TypeScript definitions are in `@zoom/videosdk/dist/types`:

- `index.d.ts` - Main exports
- `zoomvideo.d.ts` - ZoomVideo namespace
- `videoclient.d.ts` - VideoClient methods and events
- `media.d.ts` - MediaStream, audio/video options
- `common.d.ts` - Shared types (Participant, enums)
- `chat.d.ts` - Chat client
- `recording.d.ts` - Recording client
- `subsession.d.ts` - Breakout rooms

## Resources Homepage

- Official Docs: https://developers.zoom.us/docs/video-sdk/web/
- API Reference: https://marketplacefront.zoom.us/sdk/videosdk/web/
- Sample App: https://github.com/zoom/videosdk-web-sample
- Angular: https://developers.zoom.us/docs/video-sdk/web/frameworks/#using-angular
- Requirejs: https://developers.zoom.us/docs/video-sdk/web/frameworks/#using-amd-mode-requirejs-or-named-modules

## Official Docs URLs

```
# Main Documentation
https://developers.zoom.us/docs/video-sdk/web/get-started/
https://developers.zoom.us/docs/video-sdk/web/sessions/
https://developers.zoom.us/docs/video-sdk/web/event-handling/

# Audio & Video
https://developers.zoom.us/docs/video-sdk/web/video/
https://developers.zoom.us/docs/video-sdk/web/audio/
https://developers.zoom.us/docs/video-sdk/web/preview/

# Screen Sharing
https://developers.zoom.us/docs/video-sdk/web/share/
https://developers.zoom.us/docs/video-sdk/web/share-annotation/
https://developers.zoom.us/docs/video-sdk/web/share-browser-options/

# Advanced Features
https://developers.zoom.us/docs/video-sdk/web/recording/
https://developers.zoom.us/docs/video-sdk/web/subsessions/
https://developers.zoom.us/docs/video-sdk/web/live-stream/
https://developers.zoom.us/docs/video-sdk/web/command-channel/
https://developers.zoom.us/docs/video-sdk/web/transcription-translation/
https://developers.zoom.us/docs/video-sdk/web/chat/

# Phone Integration
https://developers.zoom.us/docs/video-sdk/web/pstn/
https://developers.zoom.us/docs/video-sdk/web/sip/

# Quality & Compatibility
https://developers.zoom.us/docs/video-sdk/web/quality/
https://developers.zoom.us/docs/video-sdk/web/browser-support/
https://developers.zoom.us/docs/video-sdk/web/sharedarraybuffer/
https://developers.zoom.us/docs/video-sdk/web/error-codes/

# Other
https://developers.zoom.us/docs/video-sdk/web/virtual-background/
https://developers.zoom.us/docs/video-sdk/web/frameworks/
```

## Repo-Local Appendices

The documents above are primarily based on the incoming `zoom-videosdk-web` skill. The files below remain in this repo as complementary material for deeper examples, compatibility shims, and operational debugging.

### Operational Docs

- **[RUNBOOK.md](RUNBOOK.md)** - 5-minute preflight checks before deep debugging
- **[MAINTENANCE.md](MAINTENANCE.md)** - upstream/source-maintenance notes for this imported skill

### Compatibility References

- **[references/web.md](references/web.md)** - compatibility index for repo links that previously pointed to the monolithic web reference
- **[references/web-reference.md](references/web-reference.md)** - repo-local extended API reference
- **[references/events-reference.md](references/events-reference.md)** - repo-local event catalog and payload notes
- **[troubleshooting/common-issues.md](troubleshooting/common-issues.md)** - canonical troubleshooting guide from the imported skill
- **[references/common-issues.md](references/common-issues.md)** - compatibility pointer for upstream-style reference links

### Complementary Examples

- **[examples/session-join-pattern.md](examples/session-join-pattern.md)** - complete join flow example
- **[examples/video-rendering.md](examples/video-rendering.md)** - repo-local rendering patterns
- **[examples/event-handling.md](examples/event-handling.md)** - detailed event handling examples
- **[examples/screen-share.md](examples/screen-share.md)** - screen sharing send/view patterns
- **[examples/chat.md](examples/chat.md)** - in-session messaging examples
- **[examples/command-channel.md](examples/command-channel.md)** - custom signaling examples
- **[examples/recording.md](examples/recording.md)** - recording control examples
- **[examples/transcription.md](examples/transcription.md)** - live transcription examples
- **[examples/react-hooks.md](examples/react-hooks.md)** - `@zoom/videosdk-react` patterns
- **[examples/framework-integrations.md](examples/framework-integrations.md)** - repo-local SSR and framework notes
