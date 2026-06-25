# macOS Common Issues

## Auth or IPC Never Becomes Ready

If the auth callback reports a verify-user failure, first suspect a user mismatch: the OAuth access token was issued for a different user than the user currently signed in to Zoom Workplace.

Check:

1. Zoom Workplace is installed and running.
2. The signed-in Workplace user matches the user who authorized the OAuth token.
3. The token includes `plugin_sdk:read:connection_meta`.
4. The token is not expired or revoked.
5. The OAuth redirect URL used for authorization matches the token exchange `redirect_uri`.
6. `domain` is `https://zoom.us` unless the deployment requires another documented domain.
7. The listener is retained and implements `onAuthResult` and `onIPCConnectStatusChanged`.
8. The app handles `.connectTimeout` and `.disconnected` with visible diagnostics.

To verify the mismatch, decode or introspect the OAuth token owner, check the signed-in Zoom Workplace profile, then sign out or reauthorize so both users match.

## Framework or Runtime Load Failure

- Copy the complete `PSDKLibs` dependency set, not only `ZoomToolSuite.framework`.
- Compare link, embed, search-path, and copy-phase settings with `PSDKSample.xcodeproj`.
- Re-sign every packaged framework, bundle, and dynamic library that requires the app's identity.
- Confirm the selected SDK is universal and the application architecture is supported.
- Inspect the built app bundle to ensure embedded dependencies are present.

## API Returns `false`

The request was not submitted. Check initialization, IPC state, meeting state, argument validity, and selected meeting instance before waiting for a completion that may never arrive.

## Completion Is Not Successful

Handle `ZoomToolSuiteBaseResult.error_code`:

| Error | Likely check |
|---|---|
| `.wrongUsage` | Lifecycle or meeting-state ordering |
| `.invalidArgument` | IDs, paths, enum, or missing parameter |
| `.noPermission` | Role, host policy, account setting, or OS permission |
| `.rateLimitReached` | Repeated UI actions; debounce/retry with backoff |
| `.operationTimeout` | IPC/client responsiveness and current state |
| `.versionIncompatible` | Plugin SDK and Zoom Workplace version compatibility |

## Meeting Feature Fails

- Wait for `.inMeeting`.
- Use the correct `ZoomToolSuiteMeetingInstance`.
- Resolve current user/role before host-only controls.
- Call `isSupport*`, `can*`, or state-query APIs before mutation.
- Re-read state after reconnect, promotion, demotion, waiting-room movement, or breakout-room transition.

## Share Does Not Start

- Call `canStartShare` or the feature-specific capability check.
- Verify macOS Screen Recording permission for the controlling app and Zoom Workplace.
- Verify the application/window/display identifier passed to the selected share API.
- Inspect share callbacks, including send status, content type, status change, and video-file errors.

## UI Freezes or Updates Incorrectly

Dispatch AppKit changes to the main queue:

```swift
DispatchQueue.main.async {
    // Update controls and status.
}
```

Do not perform long work in Plugin SDK callbacks.
