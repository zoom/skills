# macOS Lifecycle and Integration

Use this reference for project setup, OAuth initialization, IPC state, and orderly shutdown.

## Package Layout

The reviewed `7.1.0.595` package contains:

```text
PSDKLibs/
  ZoomToolSuite.framework
  zToolSuiteIPCProxy.framework
  supporting frameworks, bundles, and dynamic libraries
PSDKSample/
  PSDKSample.xcodeproj
  PSDKSample/*.swift
```

Treat `PSDKSample.xcodeproj` as the package-matched source of truth for framework search paths, embedded libraries, signing, and runtime dependencies.

## Integration Sequence

1. Add the full `PSDKLibs` dependency set to the Xcode project.
2. Link the required frameworks and embed the runtime frameworks, bundles, and dynamic libraries.
3. Re-sign packaged dependencies with the application's signing identity.
4. Add the `plugin_sdk:read:connection_meta` scope to the Marketplace app.
5. Obtain a user OAuth access token.
6. Create and retain a `ZoomToolSuiteEventListenerProtocol` object.
7. Call `ZoomToolSuiteAPI.initSDK`.
8. Wait for auth success and IPC connected.
9. Call pre-meeting APIs.
10. Wait for `.inMeeting` before using meeting feature namespaces.
11. Call `ZoomToolSuiteAPI.uninitSDK()` before shutdown.

## State Model

```text
uninitialized
  -> initSDK
  -> auth callback
  -> IPC connected
  -> start/join submitted
  -> connecting/waiting-room/waiting-host
  -> inMeeting
  -> disconnecting/ended
  -> uninitSDK
```

Keep application controls disabled until their required state is reached. Handle `.reconnecting`, `.failed`, `.disconnected`, and IPC timeout explicitly rather than treating the first successful connection as permanent.

## OAuth

Use a user-level OAuth token. The Plugin SDK connection requires:

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
9. Use the returned `access_token` in `ZoomToolSuiteAuthContext`.

If the app registration requires a client secret, exchange the authorization code on a trusted backend rather than shipping the secret in the app.

The redirect URL must match the Marketplace app and token request exactly. A mismatch can prevent token exchange or produce a token that cannot verify the intended Workplace user.

Chain to [Zoom OAuth](../../../oauth/SKILL.md) for token lifecycle and refresh handling.

## Operation Completion

Most feature methods:

1. return `Bool` immediately to indicate whether the request was accepted for processing;
2. invoke a completion with `ZoomToolSuiteBaseResult.error_code`;
3. may emit an event callback that reflects the resulting client state.

Do not report success from the immediate Boolean alone.

## Threading and Lifetime

- Retain the listener until after SDK uninitialization.
- Dispatch AppKit mutations to the main thread.
- Treat callback objects as snapshots unless the selected header documents a longer lifetime.
- Do not queue meeting operations while IPC is disconnected; reconcile current state after reconnection.

## Public Documentation

- [Get started](https://developers.zoom.us/docs/plugin-sdk/macos/get-started/)
- [Integrate](https://developers.zoom.us/docs/plugin-sdk/macos/integrate/)
- [Authentication](https://developers.zoom.us/docs/plugin-sdk/macos/start-join-mtg-webinar/authentication/)
- [PKCE](https://developers.zoom.us/docs/plugin-sdk/macos/start-join-mtg-webinar/pkce/)
