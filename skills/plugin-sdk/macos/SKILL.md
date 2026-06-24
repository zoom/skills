---
name: zoom-plugin-sdk-macos
description: Zoom Plugin SDK for native macOS companion applications using Swift or Objective-C to control an installed Zoom Workplace client over IPC. Use for Xcode integration, OAuth initialization, start/join flows, participant controls, audio/video settings, app or monitor sharing, recording, captions, and supported meeting UI operations on macOS.
---

# Zoom Plugin SDK for macOS

Use Swift or Objective-C to control Zoom Workplace from a separate native macOS application.

## Start Here

1. Read [Lifecycle and Integration](references/lifecycle-and-integration.md).
2. Use [Package and API Map](references/package-and-api-map.md) to locate the authoritative header and sample for a feature.
3. Implement the smallest auth and IPC connection baseline.
4. Add start/join and meeting-state handling.
5. Add one feature toolkit at a time.
6. Use [Common Issues](troubleshooting/common-issues.md) when callbacks, linking, permissions, or client compatibility fail.

## Baseline Requirements

- A Zoom Marketplace General app with Plugin SDK enabled.
- A user OAuth access token containing `plugin_sdk:read:connection_meta`.
- Zoom Workplace `7.0.2` or later according to the public setup documentation.
- Xcode `16.1` or later.
- The complete signed dependency set from the package's `PSDKLibs/` folder.

Package review baseline: `zoom-plugin-sdk-macos-universal-7.1.0.595`.

## Minimal Lifecycle

Retain the listener for the full initialized lifetime:

```swift
import Cocoa
import ZoomToolSuite

final class PluginListener: NSObject, ZoomToolSuiteEventListenerProtocol {
    func onAuthResult(_ result: ZoomToolSuiteAuthResult) {
        // Continue only after .success.
    }

    func onIPCConnectStatusChanged(
        _ status: ZoomToolSuiteIPCConnectStatus,
        errorMessage: String?
    ) {
        // Enable controls only after .connected.
    }

    func onMeetingStatusChanged(_ status: ZoomToolSuiteMeetingStatus) {
        // Enable meeting toolkits only after .inMeeting.
    }
}

final class PluginController {
    private let listener = PluginListener()

    func initialize(accessToken: String) {
        let context = ZoomToolSuiteAuthContext()
        context.accessToken = accessToken
        context.domain = "https://zoom.us"
        ZoomToolSuiteAPI.initSDK(context, listener: listener)
    }

    func shutdown() {
        ZoomToolSuiteAPI.uninitSDK()
    }
}
```

## Start or Join

```swift
let join = ZoomToolSuiteJoinMeetingParam()
join.displayName = "Display Name"
join.meetingNumber = 1234567890
join.password = ""

let submitted = ZoomToolSuiteAPI.PreMeeting.Launch.joinMeeting(param: join) { result in
    guard result.error_code == .success else {
        return
    }
}
```

`submitted` means the SDK accepted the call for processing. The completion result and subsequent meeting-status callbacks determine the actual outcome.

## Implementation Guardrails

- Wait for auth success and `.connected` IPC status before calling pre-meeting APIs.
- Wait for `.inMeeting` before calling meeting feature APIs.
- Use the relevant `ZoomToolSuiteMeetingInstance` for default meeting, green room, or breakout-room contexts.
- Check host/co-host role and feature capability before privileged operations.
- Keep UI updates on the AppKit main thread if callbacks arrive on another thread.
- Do not embed client secrets in a distributed macOS app. Use PKCE/public-client behavior or a backend token exchange appropriate to the Marketplace app configuration.
- Re-sign packaged frameworks and libraries with the application's signing identity.

## Feature Routing

| Task | API family |
|---|---|
| Start/join, settings window, URL navigation | `ZoomToolSuiteAPI.PreMeeting.Launch` |
| Status, properties, leave/end, statistics | `ZoomToolSuiteAPI.Meeting.MeetingAPI` |
| Roster, roles, permissions, host controls | `ZoomToolSuiteAPI.Meeting.Participants` |
| Mute, unmute, mute on entry | `ZoomToolSuiteAPI.Meeting.Audio` |
| Camera, pin, spotlight, avatar | `ZoomToolSuiteAPI.Meeting.Video` |
| Share application, monitor, camera, file, audio, frame, whiteboard | `ZoomToolSuiteAPI.Meeting.Share` |
| Recording, waiting room, chat/UI, captions, Q&A, webinar | Matching `ZoomToolSuiteAPI.Meeting.*` namespace |
| Device selection and audio/video processing settings | `ZoomToolSuiteAPI.Setting.Audio` and `.Video` |

## Sources

- [Official macOS docs](https://developers.zoom.us/docs/plugin-sdk/macos/)
- [Parent Plugin SDK skill](../SKILL.md)
- [Native Zoom Workplace Companion use case](../../general/use-cases/native-zoom-workplace-companion.md)
