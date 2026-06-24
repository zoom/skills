# macOS Versioning and Compatibility

## Package evidence

- SDK package: `zoom-video-sdk-macos-2.5.10.zip`
- Internal version file: `v2.5.10 (81200)`
- Changelog link is external to package contents.
- Package includes paired Markdown/JSON API docs and `Docs/skills/zm-videosdk-macos-api/SKILL.md`.

## Compatibility notes

- Keep desktop framework integration and signing settings stable per release.
- Revalidate framework linkage and app entitlements after upgrades.
- Track deprecated APIs from reference pages before jumping versions.

## Contradictions or drift to watch

- Changelog details are hosted externally; package only provides pointer links.
- Large framework bundles may include unrelated symbols; keep your surface area minimal.
