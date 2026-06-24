# macOS Reference Map

## Docs anchors

- Integration docs: https://developers.zoom.us/docs/video-sdk/macos/
- API class index: https://marketplacefront.zoom.us/sdk/custom/macos/annotated.html

## API areas to focus on

- Session context and lifecycle
- Video/audio/share helpers
- Delegate/event callback contracts
- Chat/command and auxiliary helpers

## SDK-bundled API skill (2.5.10)

- Skill: `Docs/skills/zm-videosdk-macos-api/SKILL.md`
- Index: `Docs/index.md`
- Narrative behavior: `Docs/<Module>-API.md`
- Structured signatures/examples: `Docs/<Module>-API.json`

Read these modules first when relevant:

| Area | Modules |
|------|---------|
| Lifecycle | `ZMVideoSDK`, `ZMVideoSDKDelegate`, `ZMVideoSDKDef` |
| RTMS control | `ZMVideoSDKRTMSHelper` |
| Broadcast | `ZMVideoSDKBroadcastStreamingController`, `ZMVideoSDKBroadcastStreamingViewer` |
| Network and authentication | `ZMVideoSDKNetworkConnectionHelper`, `ZMVideoSDKPasswordHandler` |
| Consent and settings | `ZMVideoSDKRecordingConsentHandler`, `ZMVideoSDKAudioSettingHelper`, `ZMVideoSDKVideoSettingHelper`, `ZMVideoSDKShareSettingHelper` |
| Collaboration | `ZMVideoSDKWhiteboardHelper`, `ZMVideoSDKSubsessionHelper`, `ZMVideoSDKAnnotationHelper` |

Cross-cutting rules from this package:

- Use delegate callbacks as the final authority for asynchronous operation completion.
- Re-check host/manager capabilities immediately after role-change callbacks.
- Treat callback handlers and SDK-owned objects as scoped unless the module explicitly permits retention.
- Handle proxy credentials and SSL verification through their callbacks rather than blind retries.

## Crawl summary

- Reference pages crawled: 242
- Docs pages crawled: 21 (20 markdown files persisted)
