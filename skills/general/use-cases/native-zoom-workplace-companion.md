# Native Zoom Workplace Companion

Build a macOS or Windows desktop application that controls the user's installed Zoom Workplace client without embedding a meeting client.

## Use This Architecture When

- A CRM, legal presentation tool, productivity app, or device console needs a **Start/Join Zoom** button.
- The native app needs to mute/unmute, control video, inspect participants, share a known application or monitor, or invoke supported Zoom Workplace UI actions.
- Users should continue using the standard Zoom Workplace client and receive its independent updates.

Do not use this architecture for headless bots, custom media rendering, or a meeting embedded inside the application's own window.

## Skill Chain

1. [Plugin SDK](../../plugin-sdk/SKILL.md)
2. Select [macOS](../../plugin-sdk/macos/SKILL.md) or [Windows](../../plugin-sdk/windows/SKILL.md)
3. [OAuth](../../oauth/SKILL.md)
4. Optional [REST API](../../rest-api/SKILL.md) for meeting creation or lookup
5. Optional [Webhooks](../../webhooks/SKILL.md) for backend lifecycle processing

Capability checkpoint: read [Plugin SDK Capabilities](../../plugin-sdk/capabilities.md) before promising that the official Zoom Workplace client can perform a requested operation.

Ambiguity checkpoint: read [Plugin SDK FAQ](../../plugin-sdk/faq.md) when comparing this architecture with Meeting SDK for Windows or macOS.

## Architecture

```text
Trusted backend (optional)
  -> create/find meeting through REST
  -> support OAuth code exchange/refresh when required

Native desktop app
  -> generate PKCE code verifier/challenge
  -> user OAuth authorization
  -> redirect URL receives authorization code
  -> token exchange sends code verifier
  -> token exchange returns access token
  -> Plugin SDK initialization
  -> local IPC
  -> installed Zoom Workplace
  -> start/join and supported meeting controls
```

## Implementation Flow

1. Enable Plugin SDK on a Marketplace General app.
2. Configure a Marketplace redirect URL.
3. Generate PKCE `code_verifier` and `code_challenge`.
4. Authorize the local user with `plugin_sdk:read:connection_meta`.
5. Receive the authorization code on the redirect URL.
6. Exchange the code for an access token with the same `redirect_uri` and original `code_verifier`.
7. Initialize the platform SDK and retain the event listener.
8. Wait for auth success and IPC connected.
9. If necessary, ask a backend to create or retrieve the meeting.
10. Submit start/join through the Plugin SDK.
11. Wait for the in-meeting state.
12. Resolve the current meeting instance, user, and role.
13. Check the capability inventory and platform API map for the requested control.
14. Enable only controls supported by the current state, role, meeting type, host policy, and client version.
15. Confirm every mutation using its completion and relevant state callback.
16. Disable controls on disconnect/reconnect and reconcile state after recovery.
17. Uninitialize on application shutdown.

## Share a Known Application Window

Use this flow for a **Share to Zoom** feature:

1. Identify the application's presentation window using native OS APIs.
2. Convert that window to the source identifier expected by the selected package.
3. Verify sharing is currently allowed.
4. Call the platform application-share API.
5. Confirm start through the completion and share-status callbacks.
6. Preserve a user-visible stop-sharing control and handle Zoom Workplace stopping share independently.

Windows exposes `StartAppShare(void* appID, ...)`; macOS exposes `startAppShare`. Verify the identifier representation in the package sample. A window title alone is not the share payload.

## Reliability Rules

- Treat the installed Zoom Workplace process as an external dependency.
- Display auth, IPC, meeting, and feature-operation states separately.
- Expect token expiry, user sign-out, client restart, policy changes, and meeting role changes.
- Never infer success from an immediate Boolean submission result.
- Use the same redirect URL for authorization and token exchange.
- Do not store refresh tokens in plaintext.
- Do not ship confidential OAuth client secrets in desktop binaries.

## Platform References

- [Plugin SDK capabilities](../../plugin-sdk/capabilities.md)
- [Plugin SDK FAQ](../../plugin-sdk/faq.md)
- [macOS capabilities](../../plugin-sdk/macos/capabilities.md)
- [macOS lifecycle](../../plugin-sdk/macos/references/lifecycle-and-integration.md)
- [macOS API map](../../plugin-sdk/macos/references/package-and-api-map.md)
- [Windows capabilities](../../plugin-sdk/windows/capabilities.md)
- [Windows lifecycle](../../plugin-sdk/windows/references/lifecycle-and-integration.md)
- [Windows API map](../../plugin-sdk/windows/references/package-and-api-map.md)
