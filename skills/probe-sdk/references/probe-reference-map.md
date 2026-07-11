# Probe SDK Reference Map

## Canonical Documentation

- Product docs: https://developers.zoom.us/docs/probe-sdk/
- Get started: https://developers.zoom.us/docs/probe-sdk/get-started/
- API reference: https://marketplacefront.zoom.us/sdk/probe/index.html

## Core Classes

- `Prober`
- `Reporter`

## Core Methods

From global/class reference surfaces:
- `requestMediaDevicePermission`
- `requestMediaDevices`
- `diagnoseAudio`
- `diagnoseVideo`
- `diagnoseScreenShare`
- `startCameraDump`
- `startToDiagnose`
- `stopToDiagnose`
- `stopToDiagnoseVideo`
- `releaseMediaStream`
- `reportBasicInfo`
- `reportFeatures`
- `cleanup`

## Evidence-Capture Guardrails

- `diagnoseScreenShare` performs environment, permission-capture, and black-frame checks. It
  requires HTTPS and is not supported on mobile browsers.
- `startCameraDump` is Chrome/Chromium-only because it requires `MediaStreamTrackProcessor`.
- Obtain explicit user consent before camera dump because the bundle includes recorded camera
  content. Stop the session before calling `downloadBundle()`.
- Treat `.probe.tar` and `.probe.tar.gz` as sensitive diagnostic evidence. Probe SDK does not
  upload them; the application or user controls transfer and retention.

## Key Enums/Constants

- `RENDERER_TYPE`
- `NETWORK_QUALITY_LEVEL`
- `BANDWIDTH_QUALITY_LEVEL`
- `PROTOCOL_TYPE`
- `ERR_CODE`
- `SUPPORTED_FEATURE_INDEX`
- `BASIC_INFO_ATTR_INDEX`

## Changelog and Package

- Changelog: https://developers.zoom.us/changelog/probe-sdk/
- npm package: https://www.npmjs.com/package/@zoom/probesdk
- sample source: https://github.com/zoom/probesdk-web
- Verified package baseline: `1.0.4` on 2026-07-10
