# macOS Versioning and Compatibility

## Observed Versions

- Latest public changelog release verified on 2026-07-10: `7.1.0`
- Local API/package inventory used by this skill: `v6.7.6.75900`
- Docs baseline: current macOS Meeting SDK docs tree captured on 2026-07-10.

The local package inventory is a historical API snapshot. Version `7.1.0` includes breaking
changes, removes `zMacResRetina.bundle`, adds I420 Limited/Full, breakout-room AI Companion,
face ROI in raw video, on-demand avatar download, and a large-meeting freeze fix. Diff the
downloaded package and deployment bundle before upgrading.

## Compatibility Practices

- Pin exact SDK package and re-test controller/delegate contracts on upgrade.
- Verify custom UI features (annotation/share/immersive) first during upgrade testing.
- Maintain a release checklist for host-only and webinar-specific flows.

## Contradiction/Drift Notes

- Docs contain both top-level and nested advanced-feature paths; treat them as parallel documentation organization, not separate APIs.
- Changelog in package is external-link based; maintain local upgrade notes for exact behavior changes.
