# Windows Lifecycle and Integration

Use this reference for Visual Studio setup, OAuth initialization, IPC state, and shutdown.

## Package Layout

The reviewed `7.1.0.2020` package provides separate `x86/` and `x64/` trees:

```text
<arch>/
  export_h/
  lib/zToolSuiteIPCProxy.lib
  bin/zToolSuiteIPCProxy.dll
  bin/other runtime dependencies
  demo/PSDKTest.sln
  demo/PSDKTest/
```

Use one architecture consistently. Do not mix x86 headers/libraries/DLLs with an x64 executable or vice versa.

## Visual Studio Setup

Configure the application with:

- include directory: `<arch>/export_h`
- library directory: `<arch>/lib`
- linker dependency: `zToolSuiteIPCProxy.lib`
- runtime files: the complete matching `<arch>/bin` contents beside the executable

The bundled project uses Visual Studio 2019 toolset (`v142`) and Windows 10 SDK settings. A newer Visual Studio can retarget the project, but validate the package sample before changing toolset or runtime assumptions.

## Integration Sequence

1. Call `InitZMToolSuite()`.
2. Build `ZMToolSuiteProxyAuthContext`.
3. Call `StartToolSuiteAuth(context)`.
4. Register a retained `IZMToolSuiteProxyListener` immediately.
5. Wait for `AUTH_RESULT_SUCCESS`.
6. Wait for `IPC_STATUS_CONNECTED`.
7. Call `premeeting::GetPreMeetingToolkit()` start/join operations.
8. Wait for `MEETING_STATUS_INMEETING`.
9. Use feature toolkits for the selected meeting instance.
10. Call `UninitZMToolSuite()` and remove the listener during shutdown.

Mirror the selected package's `PSDKTestDlg.cpp` ordering if a future package changes this sequence.

## Listener Contract

`IZMToolSuiteProxyListener` contains pure virtual methods. Compile against the selected `ToolSuiteProxyInterface.h` and implement every method required by that version.

Keep the listener alive for all callbacks. Do not destroy it while a request or IPC connection can still reference it.

## State Model

```text
uninitialized
  -> runtime initialized
  -> auth submitted
  -> auth callback
  -> IPC connected
  -> start/join submitted
  -> connecting/waiting-room/waiting-host
  -> in meeting
  -> reconnecting/disconnecting/ended
  -> runtime uninitialized
```

Disable feature controls when IPC disconnects or the meeting leaves an applicable state.

## Operation Completion

Most toolkit methods:

1. return `bool` for request submission;
2. invoke a `std::function` completion carrying `BaseResult` or a typed result;
3. may emit listener callbacks reflecting state changes.

Do not treat `true` as completed success.

## OAuth

Use a user OAuth token containing:

```text
plugin_sdk:read:connection_meta
```

Obtain the token through the OAuth authorization-code redirect flow. Prefer PKCE for native desktop apps:

1. Register a redirect URL in the Marketplace app.
2. Generate a high-entropy `code_verifier` and store it only for the current auth attempt.
3. Derive `code_challenge = BASE64URL(SHA256(code_verifier))`.
4. Start authorization with that redirect URL, required scopes, `code_challenge`, and `code_challenge_method=S256`.
5. Receive `code` on the redirect URL callback.
6. Exchange the code at `https://zoom.us/oauth/token` with `grant_type=authorization_code`.
7. Send the same `redirect_uri` value used in the authorization request.
8. Send the original `code_verifier`.
9. Use the returned `access_token` in `ZMToolSuiteProxyAuthContext`.

Do not ship a confidential OAuth client secret in the binary. If the app registration requires a secret, perform code exchange and refresh on a trusted backend.

The redirect URL must match the Marketplace app and token request exactly. A mismatch can prevent token exchange or produce a token that cannot verify the intended Workplace user.

Chain to [Zoom OAuth](../../../oauth/SKILL.md).

## Threading and Data Lifetime

- Marshal Win32/MFC UI changes to the UI thread.
- Copy callback values needed after callback return, especially strings and vector items backed by SDK-owned memory.
- Keep application state synchronized with IPC, meeting-status, user-status, and share-status callbacks.
- Avoid blocking listener callbacks.

## Public Documentation

- [Get started](https://developers.zoom.us/docs/plugin-sdk/windows/get-started/)
- [Integrate](https://developers.zoom.us/docs/plugin-sdk/windows/integrate/)
- [Authentication](https://developers.zoom.us/docs/plugin-sdk/windows/start-join-mtg-webinar/authentication/)
- [PKCE](https://developers.zoom.us/docs/plugin-sdk/windows/start-join-mtg-webinar/pkce/)
