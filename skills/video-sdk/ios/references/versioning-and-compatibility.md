# iOS Versioning and Compatibility

## Package evidence

- SDK package: `zoom-video-sdk-iOS-2.5.10.zip`
- Internal version file: `v2.5.10(35913)`
- Changelog is externally hosted via developer support portal.
- API documentation and its bundled skill are under `Sample-Libs/Docs/`.

## Compatibility notes

- Keep iOS SDK and backend token claims aligned to same release train.
- Recheck Swift/ObjC bridge and framework embedding settings on upgrade.
- Audit deprecated APIs from the iOS reference when moving minor/major versions.

## Contradictions or drift to watch

- Changelog content is not bundled in package; release details are external.
- The bundled skill assumes `Docs/` is at the selected SDK root; in this archive the correct root is `Sample-Libs/Docs/`.
