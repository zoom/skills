---
name: zoom-plugin-sdk-windows
description: Zoom Plugin SDK for native Windows companion applications using C++, Win32, or MFC to control an installed Zoom Workplace client over IPC. Use for Visual Studio integration, OAuth initialization, listener implementation, start/join flows, participant controls, audio/video settings, window or monitor sharing, recording, captions, and supported meeting UI operations on Windows.
---

# Zoom Plugin SDK for Windows

Use native C++ to control Zoom Workplace from a separate Windows desktop application.

## Start Here

1. Read [Lifecycle and Integration](references/lifecycle-and-integration.md).
2. Use [Package and API Map](references/package-and-api-map.md) to locate the authoritative interface and sample dialog for a feature.
3. Build the bundled `PSDKTest` sample in the matching architecture.
4. Implement auth, IPC, and meeting-state callbacks before feature controls.
5. Use [Common Issues](troubleshooting/common-issues.md) for build, runtime DLL, listener, permission, and compatibility failures.

## Baseline Requirements

- A Zoom Marketplace General app with Plugin SDK enabled.
- A user OAuth access token containing `plugin_sdk:read:connection_meta`.
- Zoom Workplace `7.0.2` or later according to the public setup documentation.
- Visual Studio 2019 or later.
- A consistent x86 or x64 selection across headers, library, runtime DLLs, application, and Zoom Workplace compatibility.

Package review baseline: `zoom-plugin-sdk-windows-7.1.0.2020`.

## Minimal Lifecycle

`IZMToolSuiteProxyListener` is a pure virtual interface in this package. Implement every required callback, even if most handlers are initially no-ops.

```cpp
#include "ToolSuiteProxyInterface.h"

using namespace ZMToolSuiteProxy;

class PluginController : public IZMToolSuiteProxyListener {
public:
    void Initialize(const std::wstring& accessToken) {
        InitZMToolSuite();

        ZMToolSuiteProxyAuthContext context;
        context.domain = L"https://zoom.us";
        context.accessToken = accessToken.c_str();

        StartToolSuiteAuth(context);
        SetToolSuiteProxyListener(this);
    }

    void Shutdown() {
        UninitZMToolSuite();
        SetToolSuiteProxyListener(nullptr);
    }

    void OnAuthResult(ZMToolSuiteProxyAuthResult result) override {}
    void OnIPCConnectStatusChanged(
        ZMToolSuiteIPCConnectStatus status,
        std::string errorMessage
    ) override {}
    void OnMeetingStatusChanged(ZMToolSuiteMeetingStatus status) override {}

    // Implement the remaining pure virtual callbacks from
    // ToolSuiteProxyInterface.h for the selected package version.
};
```

Keep the controller alive until after `UninitZMToolSuite()` and listener removal.

## Start or Join

```cpp
JoinMeetingParam join;
join.displayName = L"Display Name";
join.meetingNumber = 1234567890LL;
join.password = L"";

bool submitted = premeeting::GetPreMeetingToolkit()->JoinMeeting(
    join,
    [](const BaseResult& result) {
        if (result.error_code != kZMToolSuiteProxyErrorsSuccess) {
            return;
        }
    }
);
```

The Boolean reports whether the request was submitted. Use `BaseResult.error_code` and `OnMeetingStatusChanged` for the asynchronous outcome.

## Implementation Guardrails

- Wait for `AUTH_RESULT_SUCCESS` and `IPC_STATUS_CONNECTED` before pre-meeting controls.
- Wait for `MEETING_STATUS_INMEETING` before meeting toolkits.
- Use the correct `ZMToolSuiteMeetingInstance` for default, green-room, or breakout-room contexts.
- Check role, policy, and `Can*`/`IsSupport*` methods before privileged actions.
- Keep callback-owned pointers and vector data within their documented lifetime; copy data needed later.
- Marshal callback-driven UI changes to the application's UI thread.
- Ship the complete matching `bin/` runtime dependency set beside the executable.
- Do not embed an OAuth client secret in a distributed desktop binary.

## Feature Routing

| Task | API family |
|---|---|
| Initialize/authenticate/uninitialize/listener | `ToolSuiteProxyInterface.h` |
| Start/join, settings window, URL navigation | `premeeting::GetPreMeetingToolkit()` |
| Status, properties, leave/end, statistics | `meeting::GetMeetingToolkit(instance)` |
| Roster, roles, permissions, host controls | `meeting::GetParticipantsToolkit(instance)` |
| Audio/video controls | `meeting::GetAudioToolkit(instance)` and `GetVideoToolkit(instance)` |
| App/window, monitor, camera, file, audio, frame, whiteboard share | `meeting::GetShareToolkit(instance)` |
| Recording, waiting room, chat, captions, Q&A, webinar | Matching `meeting::Get*Toolkit(instance)` |
| Device and processing settings | `setting::GetSettingToolkit()` |

## Sources

- [Official Windows docs](https://developers.zoom.us/docs/plugin-sdk/windows/)
- [Parent Plugin SDK skill](../SKILL.md)
- [Native Zoom Workplace Companion use case](../../general/use-cases/native-zoom-workplace-companion.md)
