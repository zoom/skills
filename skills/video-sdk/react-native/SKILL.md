---
name: zoom-video-sdk-react-native
description: |
  Zoom Video SDK for React Native. Use when building custom mobile video session experiences
  with @zoom/react-native-videosdk, event listeners, helper-based APIs, and backend JWT token flows.
triggers:
  - react native video sdk
  - zoom react native video sdk
  - "@zoom/react-native-videosdk"
  - custom video react native
  - react native session join
  - zoom videosdk react native
---

# Zoom Video SDK (React Native)

Use this skill for React Native apps that need fully custom video session experiences using Zoom Video SDK.

## Quick Links

1. **[Lifecycle Workflow](concepts/lifecycle-workflow.md)** - init -> listeners -> join -> helpers -> leave -> cleanup
2. **[SDK Architecture Pattern](concepts/sdk-architecture-pattern.md)** - provider + helper model
3. **[High-Level Scenarios](concepts/high-level-scenarios.md)** - common mobile product patterns
4. **[Setup Guide](examples/setup-guide.md)** - package + platform setup baseline
5. **[Session Join Pattern](examples/session-join-pattern.md)** - tokenized join flow
6. **[Event Handling Pattern](examples/event-handling-pattern.md)** - event listener to state routing
7. **[SKILL.md](SKILL.md)** - complete navigation

## Core Notes

- Video SDK sessions are not Zoom Meetings and use session tokens.
- JWT generation must stay backend-side.
- Wrapper is helper-heavy (audio/video/chat/share/recording/transcription, etc.).
- Event-driven design is required for robust UI state.

## References

- [React Native Reference Index](references/react-native-reference.md)
- [Module Map](references/module-map.md)
- [Official Sources](references/official-sources.md)
- [Deprecated and Contradictions](troubleshooting/deprecated-and-contradictions.md)

## SDK-Bundled API Skill

Video SDK React Native 2.5.10 includes `docs/ai-docs/skills/zm-videosdk-react-native-api/SKILL.md`, paired `docs/ai-docs/*-API.md` / `*-API.json` files, TypeScript source under `src/`, and a working app under `example/`.

- Keep this repository skill as the workflow and troubleshooting entry point.
- Use `docs/ai-docs/index.md`, then the relevant JSON and Markdown pair, for exact wrapper signatures, event payloads, Promise behavior, and platform support.
- Cross-check `src/` before generating code. Do not invent a JavaScript API merely because the native Android or iOS SDK exposes it.
- Ignore the bundled skill's stale 2.5.7 native fallback paths; this package and its local source are 2.5.10.
- See [Module Map](references/module-map.md) for high-value modules.

## Related Skills

- [zoom-video-sdk](../SKILL.md)
- [zoom-oauth](../../oauth/SKILL.md)
- [zoom-general](../../general/SKILL.md)

## Documentation Index

### Start Here

1. [SKILL.md](SKILL.md)
2. [Lifecycle Workflow](concepts/lifecycle-workflow.md)
3. [SDK Architecture Pattern](concepts/sdk-architecture-pattern.md)
4. [Setup Guide](examples/setup-guide.md)

### Concepts

- [Lifecycle Workflow](concepts/lifecycle-workflow.md)
- [SDK Architecture Pattern](concepts/sdk-architecture-pattern.md)
- [High-Level Scenarios](concepts/high-level-scenarios.md)

### Examples

- [Setup Guide](examples/setup-guide.md)
- [Session Join Pattern](examples/session-join-pattern.md)
- [Event Handling Pattern](examples/event-handling-pattern.md)

### References

- [React Native Reference Index](references/react-native-reference.md)
- [Module Map](references/module-map.md)
- [Official Sources](references/official-sources.md)

### Troubleshooting

- [Common Issues](troubleshooting/common-issues.md)
- [Version Drift](troubleshooting/version-drift.md)
- [Deprecated and Contradictions](troubleshooting/deprecated-and-contradictions.md)

## Operations

- [RUNBOOK.md](RUNBOOK.md) - 5-minute preflight and debugging checklist.
