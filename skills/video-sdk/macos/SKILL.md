---
name: zoom-video-sdk-macos
description: |
  Zoom Video SDK for macOS native desktop apps. Use when building custom macOS video sessions
  with native UI control, tokenized join, and desktop-oriented media/device workflows.
triggers:
  - "video sdk macos"
  - "zoom video mac"
  - "macos custom video"
  - "mac desktop video sdk"
  - "video sdk cocoa"
---

# Zoom Video SDK (macOS)

Use this skill when building custom macOS desktop video session apps.

> **Current release:** `2.6.0` (verified 2026-07-10). The bundled inventory below is `2.5.10`.
> Review renamed statistic/audio APIs, framework removal, external-source changes, and new
> callbacks in [the 2.6.0 upgrade notes](../references/native-2.6.0-upgrade.md).

## Start Here

1. [macos.md](macos.md)
2. [concepts/lifecycle-workflow.md](concepts/lifecycle-workflow.md)
3. [concepts/architecture.md](concepts/architecture.md)
4. [examples/session-join-pattern.md](examples/session-join-pattern.md)
5. [scenarios/high-level-scenarios.md](scenarios/high-level-scenarios.md)
6. [references/macos-reference-map.md](references/macos-reference-map.md)
7. [references/environment-variables.md](references/environment-variables.md)
8. [references/versioning-and-compatibility.md](references/versioning-and-compatibility.md)
9. [troubleshooting/common-issues.md](troubleshooting/common-issues.md)

## SDK-Bundled API Skill

Video SDK macOS 2.5.10 includes `Docs/skills/zm-videosdk-macos-api/SKILL.md` and paired `Docs/*-API.md` / `Docs/*-API.json` references.

- Use this repository skill for architecture, implementation workflow, scenarios, and troubleshooting.
- Use the bundled skill and `Docs/index.md` to verify Objective-C signatures, asynchronous completion, role requirements, callback threading, and object lifetime.
- Prefer packaged headers over generated documentation if they conflict.
- For RTMS signaling, media sockets, protocol messages, and backend processing after `ZMVideoSDKRTMSHelper` starts the stream, chain to [zoom-rtms](../../rtms/SKILL.md).
- See [references/macos-reference-map.md](references/macos-reference-map.md) for high-value modules.

## Key Sources

- Docs: https://developers.zoom.us/docs/video-sdk/macos/
- API reference: https://marketplacefront.zoom.us/sdk/custom/macos/annotated.html
- Parent skill: [../SKILL.md](../SKILL.md)

## Operations

- [RUNBOOK.md](RUNBOOK.md) - 5-minute preflight and debugging checklist.
