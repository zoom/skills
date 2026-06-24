# iOS Reference Map

## Docs anchors

- Integration docs: https://developers.zoom.us/docs/video-sdk/ios/
- API class index: https://marketplacefront.zoom.us/sdk/custom/ios/annotated.html

## API areas to focus on

- Session lifecycle + context models
- Delegate callbacks for participant/media state
- Audio/video/share helpers
- Chat/command and advanced control surfaces

## SDK-bundled API skill (2.5.10)

- Skill: `Sample-Libs/Docs/zm-videosdk-ios-api/SKILL.md`
- Documentation root: `Sample-Libs/Docs`
- Index: `Sample-Libs/Docs/index.md`
- Objective-C sample: `Sample-Libs/ZoomVideoSample/ZoomVideoSample`
- SwiftUI sample: `Sample-Libs/ZoomVideoSwiftSample`

Read the paired JSON and Markdown files for:

| Area | Modules |
|------|---------|
| Lifecycle | `ZoomVideoSDK`, `ZoomVideoSDKDelegate`, `ZoomVideoSDKConstants` |
| RTMS control | `ZoomVideoRealTimeMediaStreamsHelper` |
| Screen sharing | `ZoomVideoSDKShareHelper`, `ZoomVideoSDKScreenShareService`, `ZoomVideoSDKShareSender`, `ZoomVideoSDKShareAudioSender` |
| Broadcast and collaboration | `ZoomVideoSDKBroadcastStreamingHelper`, `ZoomVideoSDKWhiteboardHelper`, `ZoomVideoSDKSubSessionHelper` |
| Media quality | `ZoomVideoSDKAudioSettingHelper`, `ZoomVideoSDKVideoSettingHelper`, `ZoomVideoSDKTestAudioDeviceHelper` |
| Advanced features | `ZoomVideoSDKFileTranserHandle`, `ZoomVideoSDKMaskHelper`, `ZoomVideoSDKRemoteCameraControlHelper` |

Cross-cutting rules from this package:

- Register the delegate before starting operations that complete asynchronously.
- Treat `Errors_Success` as request acceptance when a delegate callback defines final state.
- Wait for RTMS status callbacks between start, pause, resume, stop, and restart transitions.
- Export whiteboard data and wait for completion before stopping its share.
- Keep ReplayKit app-group and broadcast-extension configuration aligned between the host app and extension.

## Crawl summary

- Reference pages crawled: 324
- Docs pages crawled: 23 (22 markdown files persisted)
