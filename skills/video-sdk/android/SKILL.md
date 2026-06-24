---
name: zoom-video-sdk-android
description: |
  Zoom Video SDK for Android native apps. Use when building custom Android video experiences
  with full UI control, session tokens, raw media options, and event-driven participant state.
triggers:
  - "video sdk android"
  - "zoom video android"
  - "android custom video"
  - "android video session"
---

# Zoom Video SDK (Android)

Use this skill when building Android apps with custom real-time video sessions.

## Start Here

1. [android.md](android.md)
2. [concepts/lifecycle-workflow.md](concepts/lifecycle-workflow.md)
3. [concepts/architecture.md](concepts/architecture.md)
4. [examples/session-join-pattern.md](examples/session-join-pattern.md)
5. [scenarios/high-level-scenarios.md](scenarios/high-level-scenarios.md)
6. [references/android-reference-map.md](references/android-reference-map.md)
7. [references/environment-variables.md](references/environment-variables.md)
8. [references/versioning-and-compatibility.md](references/versioning-and-compatibility.md)
9. [troubleshooting/common-issues.md](troubleshooting/common-issues.md)

## SDK-Bundled API Skill

Video SDK Android 2.5.10 includes `Docs/skills/zm-videosdk-android-api/SKILL.md` and paired `Docs/*-API.md` / `Docs/*-API.json` references.

- Use this repository skill for architecture, implementation workflow, scenarios, and troubleshooting.
- Use the bundled skill and `Docs/index.md` for exact 2.5.10 signatures, enums, callback timing, threading, role checks, and callback-object lifetime.
- Treat package headers and exported Java APIs as authoritative if generated documentation conflicts with code.
- For RTMS signaling, media sockets, protocol messages, and backend processing after `ZoomVideoSDKRTMSHelper` starts the stream, chain to [zoom-rtms](../../rtms/SKILL.md).
- See [references/android-reference-map.md](references/android-reference-map.md) for the high-value modules and corrected archive-relative paths.

## Key Sources

- Docs: https://developers.zoom.us/docs/video-sdk/android/
- API reference: https://marketplacefront.zoom.us/sdk/custom/android/index.html
- Parent skill: [../SKILL.md](../SKILL.md)

## Operations

- [RUNBOOK.md](RUNBOOK.md) - 5-minute preflight and debugging checklist.
