# Android Versioning and Compatibility

## Observed Versions

- Latest public changelog release verified on 2026-07-10: `7.1.0`
- Local API/package inventory used by this skill: `v6.7.5.37500`
- Docs baseline: current Meeting SDK Android docs tree captured on 2026-07-10.

The local package inventory is a historical API snapshot, not a latest-version claim. Version
`7.1.0` includes breaking changes plus I420 Limited/Full, breakout-room summaries, face ROI in
raw video, and on-demand avatar download. Re-crawl/diff the exact downloaded package before using
new symbols.

## Compatibility Practices

- Pin exact SDK artifact version in Gradle.
- Track enum/interface additions between releases.
- Validate custom UI flows after each SDK update (higher break risk vs default UI).

## Contradiction/Drift Notes

- Docs contain legacy and current path variants (`add-features` vs `custom-ui/default-ui` sections).
- Some page titles include suffix artifacts (for example `| MSDK | And`), indicating doc-generation inconsistencies, not functional API differences.
