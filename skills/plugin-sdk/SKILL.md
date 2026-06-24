---
name: zoom-plugin-sdk
description: Zoom Plugin SDK guidance for native macOS and Windows applications that control an installed Zoom Workplace desktop client over IPC. Use for desktop companion apps that start or join meetings, control meeting audio/video/share/UI, inspect participants, or manage supported meeting features without embedding a Zoom client. Route platform-specific implementation to the macOS or Windows child skill.
---

# Zoom Plugin SDK

Build native desktop companion applications that control the user's installed Zoom Workplace client through inter-process communication (IPC).

## Hard Routing Guardrail

- Use Plugin SDK when a separate native macOS or Windows app must control the installed Zoom Workplace desktop client.
- Use [Meeting SDK](../meeting-sdk/SKILL.md) when Zoom meeting functionality must be embedded inside the application's own process or UI.
- Use [Zoom Apps SDK](../zoom-apps-sdk/SKILL.md) when the app runs as a web application inside Zoom Workplace.
- Use [REST API](../rest-api/SKILL.md) for server-side meeting resources, account administration, reporting, or workflows that do not control the local client.
- Do not describe Plugin SDK as a headless meeting bot or a replacement Zoom client. Zoom Workplace must be installed and compatible.

## Platform Router

| Target | Child skill |
|---|---|
| Swift, Objective-C, AppKit, Xcode, macOS | [macOS](macos/SKILL.md) |
| Native C++, Win32, MFC, Visual Studio, Windows | [Windows](windows/SKILL.md) |

Choose one child skill before writing integration code.

## Core Architecture

```text
Native companion app
  -> user OAuth access token
  -> Plugin SDK runtime
  -> local IPC connection
  -> installed Zoom Workplace client
  -> meeting/webinar UI and supported controls
```

The application does not render or transport meeting media itself. It sends supported control requests to Zoom Workplace and observes asynchronous auth, IPC, meeting, participant, media-control, sharing, and UI events.

## Common Lifecycle

1. Create a General app in Zoom App Marketplace.
2. Enable **Plugin SDK** under **Features > Access**.
3. Request user authorization with `plugin_sdk:read:connection_meta` plus scopes needed by any REST API calls.
4. Install and sign the platform SDK libraries with the application.
5. Register and retain the platform event listener.
6. Initialize the Plugin SDK with the OAuth access token and `https://zoom.us` domain.
7. Wait for successful authentication and an IPC connected event.
8. Start or join a meeting, then wait for the in-meeting status before using meeting toolkits.
9. Treat immediate Boolean returns as request-submission results and completion callbacks as operation results.
10. Uninitialize the SDK during orderly application shutdown.

## Feature Surface

The 7.1.0 packages expose platform-equivalent toolkits for:

- start/join, meeting status, meeting properties, leave/end, and statistics
- participants, roles, host/co-host controls, waiting room, and permissions
- audio/video devices and in-meeting audio/video controls
- application, monitor, camera, file, audio, frame, and whiteboard sharing
- recording, chat, reactions, closed captions/live transcription, Q&A, and webinars
- annotation, Zoom Workplace view/UI actions, and settings

Feature availability still depends on client version, meeting type, account settings, role, and host policy. Call capability checks where the SDK provides them.

## Skill Chaining

| Need | Chain to |
|---|---|
| User OAuth, PKCE, token refresh, and scopes | [zoom-oauth](../oauth/SKILL.md) |
| Create or manage meetings before local client control | [zoom-rest-api](../rest-api/SKILL.md) |
| React to server-side lifecycle events when the desktop app is offline | [zoom-webhooks](../webhooks/SKILL.md) |
| Embed a meeting instead of controlling Zoom Workplace | [zoom-meeting-sdk](../meeting-sdk/SKILL.md) |

Use [Native Zoom Workplace Companion](../general/use-cases/native-zoom-workplace-companion.md) for the complete cross-platform workflow.

## Version and Source Policy

- Public docs used for this skill: [macOS](https://developers.zoom.us/docs/plugin-sdk/macos/) and [Windows](https://developers.zoom.us/docs/plugin-sdk/windows/).
- Package review baseline: macOS `7.1.0.595` and Windows `7.1.0.2020`.
- Verify exact names and signatures against the headers and sample bundled with the downloaded package.
- Prefer package headers over examples when generated docs and the selected package disagree.
- Do not place machine-local extraction paths in implementation guidance.
