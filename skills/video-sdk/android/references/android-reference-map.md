# Android Reference Map

## Docs anchors

- Getting started and integration: https://developers.zoom.us/docs/video-sdk/android/
- API surface index: https://marketplacefront.zoom.us/sdk/custom/android/index.html

## API areas to focus on

- Session lifecycle and join context
- Audio/video helpers
- Participant/user helpers
- Share/chat/command channels
- Raw data interfaces and delegates

## SDK-bundled API skill (2.5.10)

- Skill: `Docs/skills/zm-videosdk-android-api/SKILL.md`
- Index: `Docs/index.md`
- Narrative behavior: `Docs/<Module>-API.md`
- Structured signatures/examples: `Docs/<Module>-API.json`

The bundled skill contains stale `sdk-client/docs/api/android/videosdk/...` examples. Resolve all references against the package's actual `Docs/` directory.

Read these modules first when relevant:

| Area | Modules |
|------|---------|
| Lifecycle and callbacks | `ZoomVideoSDK`, `ZoomVideoSDKDelegate`, `ZoomVideoSDKErrors` |
| RTMS control | `ZoomVideoSDKRTMSHelper`, `RealTimeMediaStreamsStatus`, `RealTimeMediaStreamsFailReason` |
| Broadcast and incoming streams | `ZoomVideoSDKBroadcastStreamingController`, `ZoomVideoSDKBroadcastStreamingViewer`, `ZoomVideoSDKIncomingLiveStreamHelper` |
| Share and media injection | `ZoomVideoSDKShareHelper`, `ZoomVideoSDKSharePreprocessParam`, `ZoomVideoSDKVideoSource`, `ZoomVideoSDKVirtualAudioMic` |
| Collaboration | `ZoomVideoSDKWhiteboardHelper`, `ZoomVideoSDKAnnotationHelper`, `ZoomVideoSDKSubSessionHelper` |
| Reliability | `ZoomVideoSDKTestAudioDeviceHelper`, `ZoomVideoSDKRecordingConsentHandler`, `ZoomVideoSDKPasswordHandler`, `ZoomVideoSDKQOSStatistics` |
| Transfer and advanced video | `ZoomVideoSDKSendFile`, `ZoomVideoSDKReceiveFile`, `ZoomVideoSDKMaskHelper`, `ZoomVideoSDKMultiCameraStreamStatus` |

Cross-cutting rules from this package:

- Call initialization and control APIs on the required main thread unless a raw-data callback explicitly documents a worker thread.
- Register listeners before `joinSession`.
- Treat success as request acceptance when a delegate callback reports final completion.
- Treat callback objects as callback-scoped unless the API documents a retention mechanism.
- Re-check role-gated helpers and actions after host or manager changes.

## Crawl summary

- Reference pages crawled: 650
- Docs pages crawled: 23 (22 markdown files persisted)
