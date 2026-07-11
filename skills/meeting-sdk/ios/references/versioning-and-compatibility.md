# iOS Versioning and Compatibility

## Observed Versions

- Latest public changelog release verified on 2026-07-10: `7.1.0`
- Local API/package inventory used by this skill: `v6.7.5.33005`
- Docs baseline: current iOS Meeting SDK docs tree captured on 2026-07-10.

The local package inventory is a historical API snapshot. Version `7.1.0` includes breaking
changes, I420 Limited/Full, breakout-room summaries, face ROI in raw video, on-demand avatar
download, and a webinar active-share unsubscribe fix. Verify declarations in the downloaded
`7.1.0` package before implementation.

## Compatibility Practices

- Pin exact SDK package release.
- Re-verify category/protocol method availability on upgrades.
- Re-test background/audio interruption flows every upgrade.

## Contradiction/Drift Notes

- Package `README.md` points to generic Meeting SDK docs root, while platform docs live at `/meeting-sdk/ios/`.
- Package changelog file only links externally; keep your own integration delta notes per release.
