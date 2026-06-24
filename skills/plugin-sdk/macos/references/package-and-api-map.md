# macOS Package and API Map

Use the headers and Swift sample bundled with the selected SDK version to verify exact signatures and availability.

## Authoritative Entry Points

| Concern | Package path |
|---|---|
| Umbrella framework | `PSDKLibs/ZoomToolSuite.framework` |
| Main init/uninit API | `ZoomToolSuite.framework/Headers/ZoomToolSuiteInterfaceObjc.h` |
| Events | `ZoomToolSuite.framework/Headers/ZoomToolSuiteEventListenerProtocol.h` |
| Shared enums/results/models | `ZoomToolSuite.framework/Headers/ZoomToolSuiteDefinesObjc.h` |
| Swift-imported surface | `ZoomToolSuite.framework/Headers/ZoomToolSuite-Swift.h` |
| Integration project | `PSDKSample/PSDKSample.xcodeproj` |
| Initialization sample | `PSDKSample/PSDKSample/AppDelegate.swift` |
| Event sample | `PSDKSample/PSDKSample/EventListener.swift` |

## Feature Map

| Feature | Header | Sample |
|---|---|---|
| Start/join and client navigation | `ZoomToolSuiteLaunchObjc.h` | `PreMeetingController.swift` |
| Meeting status/properties/statistics | `ZoomToolSuiteMeetingObjc.h` | `MeetingController.swift` |
| Participants, roles, permissions | `ZoomToolSuiteParticipantsObjc.h` | `ParticipantsController.swift` |
| In-meeting audio | `ZoomToolSuiteAudioObjc.h` | `AudioController.swift` |
| In-meeting video | `ZoomToolSuiteVideoObjc.h` | `VideoController.swift` |
| Audio/video device settings | `ZoomToolSuiteSettingObjc.h` | `AudioSettingController.swift`, `VideoSettingController.swift` |
| Sharing | `ZoomToolSuiteShareObjc.h` | `ShareController.swift` |
| Recording | `ZoomToolSuiteRecordingObjc.h` | `RecordingController.swift` |
| Waiting room | `ZoomToolSuiteWaitingRoomObjc.h` | `WaitingRoomController.swift` |
| Closed captions/live transcription | `ZoomToolSuiteClosedCaptionObjc.h` | `ClosedCaptionController.swift` |
| Reactions | `ZoomToolSuiteReactionObjc.h` | `ReactionController.swift` |
| Q&A | `ZoomToolSuiteQAObjc.h` | `QAController.swift` |
| Webinar controls | `ZoomToolSuiteWebinarObjc.h` | `WebinarController.swift` |
| Annotation | `ZoomToolSuiteAnnotationObjc.h` | `AnnotationController.swift` |
| Meeting UI actions and chat state | `ZoomToolSuiteUIActionObjc.h` | `UIActionController.swift` |

## Share Source Selection

The package exposes:

- `startAppShare`
- `startMonitorShare`
- `startCameraShare`
- `startVideoFileShare`
- `startAudioShare`
- `startFrameShare`
- `startWhiteBoardShare`

Call the relevant capability check first when one exists, obtain the source identifier using macOS APIs or the application's own known window/display context, and confirm the expected identifier type against `ZoomToolSuiteShareObjc.h` and `ShareController.swift`.

## Version Drift

- Public web docs currently document a smaller surface than the package.
- Do not infer a method from the Windows package; verify the Swift/Objective-C header.
- When upgrading, diff `ZoomToolSuite.framework/Headers/` and the sample controllers before changing implementation guidance.
- `ZoomToolSuiteErrors.versionIncompatible` indicates a package/client compatibility problem.
