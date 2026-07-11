# Android Versioning and Compatibility

## Package evidence

- Latest public changelog release: `2.6.0` (verified 2026-07-10)
- SDK package: `zoom-video-sdk-android-2.5.10.zip`
- Internal version: `v2.5.10 (40300)`
- Package includes `mobilertc.aar`, optional feature AARs, samples, and `Docs/` API documentation.

The package evidence below is a historical `2.5.10` snapshot. See
[native 2.6.0 upgrade](../../references/native-2.6.0-upgrade.md) before treating any declaration
as current.

## Compatibility notes

- Keep app and backend token logic aligned with the same Video SDK release family.
- Expect method additions/renames across releases; pin SDK version per release train.
- Revalidate proguard/R8 rules and permissions whenever upgrading.

## Contradictions or drift to watch

- Changelog points to external support portal pages, not in-package detailed notes.
- Crawl of docs subpages may miss dynamic pages; confirm against official docs at build time.
