# Rivet Versioning and Compatibility

Current verified npm release: `@zoom/rivet@0.4.0` (2026-07-10).

## Upgrade Strategy

Use the repository-standard workflow:
- [../../general/references/sdk-upgrade-workflow.md](../../general/references/sdk-upgrade-workflow.md)

For Rivet, treat upgrades as three parallel checks:
1. `@zoom/rivet` package release changes
2. Underlying Zoom API/event payload changes
3. Marketplace app config and scope changes

## Compatibility Risks

- Module/auth behavior drift across versions.
- Type alias or endpoint wrapper signature changes.
- Event payload shape differences for webhook types.
- Receiver behavior changes in Node runtime/Lambda environments.

## 0.4.0 Breaking Changes

- `CommerceOAuthClient` became `CommerceS2SAuthClient` and requires S2S credentials.
- Meetings renamed `upgradeZpaOsApp` to `upgradeZPAFirmwareOrApp`.
- Meetings renamed `listMeetingSummariesOfAccount` to `listAccountsMeetingOrWebinarSummaries`.
- Meetings renamed `getMeetingSummary` to `getMeetingOrWebinarSummary`.
- Phone renamed `addPolicySettingToCallQueue` to `addPolicySubsettingToCallQueue`.
- Phone renamed `addCommonAreaSettings` / `updateCommonAreaSettings` to singular names.
- Team Chat renamed `getChatMessagesReports` to `getChatMessageReports`.
- Users renamed `bulkUpdateFeaturesForUser` to `bulkUpdateFeaturesForUsers`.

Search and replace only after compiling against `0.4.0`; generated endpoint names can change
again when the underlying OpenAPI clients are regenerated.

## Version Signals

- `@zoom/rivet` package version in `package.json`.
- TypeDoc module/classes/type aliases under `zoom.github.io/rivet-javascript`.
- Changelog updates under https://developers.zoom.us/changelog/.

## Safe Upgrade Checklist

- Pin current and target `@zoom/rivet` versions.
- Compare TypeDoc module pages for changed constructor options and endpoints.
- Validate event names and payload fields used by your handlers.
- Revalidate OAuth install/callback flow and token persistence.
- Revalidate per-module port and `/zoom/events` mapping.
