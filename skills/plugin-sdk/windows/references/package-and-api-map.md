# Windows Package and API Map

Use the selected architecture's headers and MFC sample to verify exact signatures and behavior.

## Authoritative Entry Points

| Concern | Package path |
|---|---|
| Initialization and toolkit factories | `<arch>/export_h/ToolSuiteProxyInterface.h` |
| Enums, models, parameters, results | `<arch>/export_h/ToolSuiteProxyDef.h` |
| Link library | `<arch>/lib/zToolSuiteIPCProxy.lib` |
| Runtime proxy | `<arch>/bin/zToolSuiteIPCProxy.dll` |
| Build sample | `<arch>/demo/PSDKTest.sln` |
| Init/events sample | `<arch>/demo/PSDKTest/PSDKTestDlg.cpp` and `.h` |

## Feature Map

| Feature | Interface | Sample |
|---|---|---|
| Start/join and client navigation | `ToolSuitePreMeetingInterface.h` | `PreMeetingTestDlg.cpp` |
| Meeting status/properties/statistics | `ToolSuiteMeetingInterface.h` | `MeetingTestDlg.cpp`, `StatisticTestDlg.cpp` |
| Participants, roles, permissions | `ToolSuiteParticipantsInterface.h` | `ParticipantTestDialog.cpp` |
| In-meeting audio | `ToolSuiteAudioInterface.h` | `AudioTestDlg.cpp` |
| In-meeting video | `ToolSuiteVideoInterface.h` | `VideoTestDlg.cpp` |
| Device and media settings | `ToolSuiteSettingInterface.h` | `SettingDlg.cpp` |
| Sharing | `ToolSuiteShareInterface.h` | `ShareTestDlg.cpp` |
| Recording | `ToolSuiteRecordingInterface.h` | `RecordingTestDlg.cpp` |
| Waiting room | `ToolSuiteWaitingRoomInterface.h` | `WaitingRoomTestDlg.cpp` |
| Chat | `ToolSuiteChatInterface.h` | `ChatDlg.cpp` |
| Closed captions/live transcription | `ToolSuiteClosedCaptionInterface.h` | Main test dialog integration |
| Reactions | `ToolSuiteReactionInterface.h` | `ReactionTestDlg.cpp` |
| Q&A | `ToolSuiteQAInterface.h` | `QATestDlg.cpp` |
| Webinar controls | `ToolSuiteWebinarInterface.h` | `WebinarTestDlg.cpp` |
| Annotation | `ToolSuiteAnnotationInterface.h` | `AnnotationTestDlg.cpp` |
| Zoom Workplace UI actions | `ToolSuiteUIActionInterface.h` | `UIActionTestDlg.cpp` |

## Application or Window Sharing

`IShareToolkit` exposes:

```cpp
bool StartAppShare(void* appID, onStartAppShareResult callback);
bool StartMonitorShare(void* monitorID, onStartMonitorShareResult callback);
```

It also supports camera, video-file, audio, frame, and whiteboard sharing. Obtain the platform-native source identifier using the same representation expected by the selected package and verify it against `ShareTestDlg.cpp`; do not pass a window title string directly unless the package explicitly adds such an API.

Call `CanStartShare` before sharing and use listener callbacks to confirm that sharing actually started or stopped.

## Meeting Instances

Factories generally accept:

- `ZMToolSuiteMeetingInstance_Default`
- `ZMToolSuiteMeetingInstance_GreenRoom`
- `ZMToolSuiteMeetingInstance_NewBO`

Use the instance matching the current context. A valid user ID from one context should not be assumed valid in another.

## Version Drift

- Public web docs cover fewer features than the package headers.
- x86 and x64 headers in the reviewed package are expected to represent the same API, but use the files from the application architecture.
- Diff `export_h/` and `demo/PSDKTest/` during upgrades.
- `kZMToolSuiteProxyErrorsVersionIncompatible` indicates a package/client compatibility problem.
