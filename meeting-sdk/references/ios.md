# Meeting SDK (iOS) Pointers

This repo focuses heavily on Web/Linux/Windows. This file exists to avoid dead links and to provide a consistent "where do I start" pointer for iOS Meeting SDK questions.

Primary skill entrypoint:
- [../ios/SKILL.md](../ios/SKILL.md)

## Common Forum Questions

- How do I initialize the iOS Meeting SDK?
- What permissions/entitlements do I need for camera/mic?
- How do I customize UI vs default UI?

## Practical Guidance

- Start with default UI until basic join/start works.
- Confirm camera/mic permission flows.
- When debugging join failures, capture:
  - SDK version
  - init/join return codes
  - meeting number vs meeting UUID confusion
- Use [../ios/references/ios-reference-map.md](../ios/references/ios-reference-map.md) to confirm current API surface before assuming wrapper/sample parity.

## Next

- `android.md` for Android equivalents.
- `authorization.md` and `signature-playbook.md` for auth/signature concepts shared across platforms.
