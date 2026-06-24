---
name: zoom-video-sdk-ios
description: |
  Zoom Video SDK for iOS native apps. Use when building custom iOS video sessions with
  full UI control, token-based session auth, and event-driven media/participant flows.
triggers:
  - "video sdk ios"
  - "zoom video ios"
  - "ios custom video"
  - "xcframework video sdk"
  - "ios video session"
---

# Zoom Video SDK (iOS)

Use this skill when building custom iOS video session experiences.

## Start Here

1. [ios.md](ios.md)
2. [concepts/lifecycle-workflow.md](concepts/lifecycle-workflow.md)
3. [concepts/architecture.md](concepts/architecture.md)
4. [examples/session-join-pattern.md](examples/session-join-pattern.md)
5. [scenarios/high-level-scenarios.md](scenarios/high-level-scenarios.md)
6. [references/ios-reference-map.md](references/ios-reference-map.md)
7. [references/environment-variables.md](references/environment-variables.md)
8. [references/versioning-and-compatibility.md](references/versioning-and-compatibility.md)
9. [troubleshooting/common-issues.md](troubleshooting/common-issues.md)

## SDK-Bundled API Skill

Video SDK iOS 2.5.10 includes `Sample-Libs/Docs/zm-videosdk-ios-api/SKILL.md` and paired `Sample-Libs/Docs/*-API.md` / `Sample-Libs/Docs/*-API.json` references.

- Use this repository skill for architecture, implementation workflow, scenarios, and troubleshooting.
- Resolve the bundled documentation root as `Sample-Libs/Docs`, not the archive root assumed by the unadapted bundled instructions.
- Use the bundled docs for exact Objective-C signatures, delegate completion, role checks, state machines, and object lifetime; use the packaged headers as final authority.
- For RTMS signaling, media sockets, protocol messages, and backend processing after the native helper starts the stream, chain to [zoom-rtms](../../rtms/SKILL.md).
- See [references/ios-reference-map.md](references/ios-reference-map.md) for high-value modules and sample locations.

## Key Sources

- Docs: https://developers.zoom.us/docs/video-sdk/ios/
- API reference: https://marketplacefront.zoom.us/sdk/custom/ios/annotated.html
- Parent skill: [../SKILL.md](../SKILL.md)

## Operations

- [RUNBOOK.md](RUNBOOK.md) - 5-minute preflight and debugging checklist.
