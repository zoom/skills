# Unity Versioning and Compatibility

## Package evidence

- Latest public wrapper changelog release: `0.0.2` (verified 2026-07-10)
- Wrapper package: `unity-zoom-video-sdk-0.0.2-beta.zip`
- Contains `ZoomVideoSDK.unitypackage` wrapper bundle.

## Compatibility notes

- Unity wrapper versioning is independent from native Android/iOS/macOS SDK streams.
- Validate each API/event used in wrapper reference docs before implementation.
- Treat wrapper as potentially partial feature surface versus native SDKs.

## Contradictions or drift to watch

- Wrapper release (`0.0.2-beta`) is substantially behind the native `2.6.0` release family.
- Some docs and feature names may differ between wrapper and native platform references.
