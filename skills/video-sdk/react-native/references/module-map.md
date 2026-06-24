# Module Map

## Core wrapper

- `native/ZoomVideoSdk`
- provider/hooks modules (`ZoomVideoSdkProvider`, `hooks/useZoom`, `hooks/useSdkHandler`, `hooks/useSdkEventListener`)

## SDK-bundled API skill (2.5.10)

- Skill: `docs/ai-docs/skills/zm-videosdk-react-native-api/SKILL.md`
- Index: `docs/ai-docs/index.md`
- API pairs: `docs/ai-docs/<Module>-API.md` and `docs/ai-docs/<Module>-API.json`
- TypeScript authority: `src/`
- Working integration examples: `example/src/`

Read the index first, then the JSON for signatures and the Markdown for lifecycle guidance. Cross-check public exports and validation in `src/`; native iOS/Android APIs do not imply React Native wrapper support.

## Helpers

- session/user helpers
- audio/video/share helpers
- chat + command channel
- recording/live stream/live transcription
- phone/CRC/annotation/subsession helpers
- broadcast streaming controller/viewer
- whiteboard and `WhiteboardView`
- pre-session audio testing
- audio settings and remote camera control

## Types and enums

- join/init config types
- event enum/type map
- error/status/permission/resolution enums

## High-value 2.5.10 modules

| Area | Modules |
|------|---------|
| Provider and lifecycle | `ZoomVideoSdkProvider`, `ZoomVideoSdk`, `useZoom`, `useSdkEventListener`, `EventType` |
| Rendering | `ZoomView`, `CameraView`, `WhiteboardView`, `BroadcastStreamingView` |
| Broadcast | `ZoomVideoSdkBroadcastStreamingHelper`, `ZoomVideoSdkBroadcastStreamingViewer` |
| Collaboration | `ZoomVideoSdkWhiteboardHelper`, `ZoomVideoSdkAnnotationHelper`, `ZoomVideoSdkSubSession` |
| Reliability | `ZoomVideoSdkTestAudioDeviceHelper`, `ZoomVideoSdkRecordingHelper`, `Errors` |

Cross-cutting rules:

- Mount `ZoomVideoSdkProvider` before calling `useZoom`.
- Subscribe to session events before `joinSession` and remove listeners during component cleanup.
- Promise resolution may only mean the native request was accepted; use the documented `EventType` for final state.
- Event payloads are copied onto the JavaScript thread and may be retained, unlike native callback-scoped objects.
- Ignore the bundled skill's references to native SDK 2.5.7; this package's wrapper and local source are 2.5.10.
