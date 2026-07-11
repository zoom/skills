# SDK Upgrade Guide

Use this guide to establish a current version baseline before changing a Zoom SDK integration.
For a version-by-version migration plan, use
[sdk-upgrade-workflow.md](sdk-upgrade-workflow.md).

## Source Priority

Check sources in this order:

1. Installed package metadata, headers, and bundled documentation.
2. Official Zoom changelog and minimum-version pages.
3. Official package registry metadata and Zoom sample repositories.
4. Existing application behavior and regression tests.

Do not use one platform's version as the version for another platform. Native Meeting SDK,
native Video SDK, Web, React Native, Flutter, Unity, and Unreal have independent release trains.

## Public Package Snapshot

Verified on 2026-07-10:

| Product | Package | Verified version |
|---------|---------|------------------|
| Meeting SDK Web | `@zoom/meetingsdk` | `6.2.0` |
| Meeting SDK React Native | `@zoom/meetingsdk-react-native` | `7.0.5` |
| Video SDK Web | `@zoom/videosdk` | `2.4.5` |
| Video SDK React Native | `@zoom/react-native-videosdk` | `2.5.10` |
| Video SDK Flutter | `flutter_zoom_videosdk` | `2.5.10` |
| Video SDK UI Toolkit Web | `@zoom/videosdk-ui-toolkit` | `2.4.5-1` |
| Zoom Apps SDK | `@zoom/appssdk` | `0.16.39` |
| Probe SDK | `@zoom/probesdk` | `1.0.4` |
| Rivet JavaScript | `@zoom/rivet` | `0.4.0` |

This is a checked snapshot, not a permanent `latest` declaration. Query the package registry
again before pinning a new release.

## Public Changelog Snapshot

Verified on 2026-07-10:

| Product/platform | Current changelog release |
|------------------|---------------------------|
| Meeting SDK Android/Electron/iOS/Linux/macOS/Windows | `7.1.0` |
| Meeting SDK Web | `6.2.0` |
| Meeting SDK React Native | `7.0.5` |
| Meeting SDK Unreal wrapper | `1.1.0` (Meeting SDK `6.1.5`) |
| Video SDK Android/iOS/Linux/macOS/Windows | `2.6.0` |
| Video SDK Web | `2.4.5` |
| Video SDK React Native/Flutter | `2.5.10` |
| Video SDK Unity wrapper | `0.0.2` |
| Contact Center Web/Android/iOS | `5.6.0` |
| Cobrowse SDK Web | `3.6.0` |
| Plugin SDK macOS/Windows product release | `1.0.0` initial launch |

Plugin SDK's product-release label is separate from its bundled package/build number. Keep the
two values distinct when reporting compatibility.

## Native Package Baselines

Native SDKs are downloaded from Zoom Marketplace or Build Platform and must be checked per
platform. This repository's reviewed package snapshots currently include:

| Product/platform | Reviewed package baseline |
|------------------|---------------------------|
| Meeting SDK Android | `6.7.5.37500` |
| Meeting SDK iOS | `6.7.5.33005` |
| Meeting SDK macOS | `6.7.6.75900` |
| Meeting SDK Windows interface guide | `6.7.2.26830` |
| Meeting SDK Unreal | `6.1.5.43366` with UE `5.4.3` |
| Video SDK Android/iOS/macOS/Linux/Windows | `2.5.10` package family |

Treat these as the versions whose bundled headers and API skills were reviewed, not proof that
each is still the newest downloadable package.

Current public changelogs are ahead of several reviewed snapshots: native Meeting SDK is
`7.1.0`, and native Video SDK is `2.6.0`. Do not relabel an older API inventory as current;
download the new package and regenerate/diff its declarations first.

## Upgrade Workflow

1. Record exact product, platform, package, architecture, and current version.
2. Query the current package registry or download the target native package.
3. Read every changelog entry between current and target versions.
4. Diff public headers, TypeScript declarations, exported modules, and required callbacks.
5. Update dependencies without mixing package families.
6. Rebuild from clean dependency caches.
7. Run lifecycle, auth, media, event, and cleanup regression tests.
8. Record the validated version and known feature gates.

## Web Package Checks

```bash
npm view @zoom/meetingsdk version peerDependencies
npm view @zoom/videosdk version peerDependencies
npm view @zoom/meetingsdk-react-native version peerDependencies
npm view @zoom/react-native-videosdk version peerDependencies
npm view @zoom/videosdk-ui-toolkit version peerDependencies
npm view @zoom/appssdk version
npm view @zoom/probesdk version
npm view @zoom/rivet version
```

For Flutter:

```bash
curl -fsSL https://pub.dev/api/packages/flutter_zoom_videosdk | jq -r '.latest.version'
```

## Current Web Guardrails

- Meeting SDK Web Client View and Component View are parallel integration models. Do not
  describe `ZoomMtg.init()` as deprecated merely because Component View uses `client.init()`.
- `@zoom/meetingsdk@6.2.0` declares React and React DOM `18.2.0` peer dependencies. Do not claim
  React 19 support from an application-level compatibility override.
- Use Meeting SDK Web `5.1.4` or newer for WebRTC video compatibility with future Chrome
  releases.
- Meeting SDK Web is for human meeting participants. Route bots and AI notetakers to RTMS.
- UI Toolkit `2.4.5-1` expects `@zoom/videosdk@^2.4.5` and React 18 peers.
- `@zoom/appssdk` npm patch version and `config({ version: '0.16' })` protocol version are
  different values.

## Native Upgrade Checks

- Diff pure virtual interfaces before compiling Windows or Linux C++ implementations.
- Implement every callback required by the selected package, even if a callback is not used.
- Recheck deployment targets, permissions, signing, and native library architecture.
- Test default UI and custom UI separately for Meeting SDK.
- Verify wrapper-to-native version compatibility for React Native, Flutter, Unity, and Unreal.
- Use the SDK-bundled Markdown/JSON API skills when present, then verify exact signatures in
  packaged headers or source.

## Regression Matrix

At minimum, test:

| Area | Checks |
|------|--------|
| Initialization | success, repeated init, cleanup, unsupported platform |
| Authentication | valid token, expired token, wrong app/account, refresh path |
| Session lifecycle | join/start, reconnect, leave/end, app background/foreground |
| Media | camera, microphone, speaker, share, device changes, permissions |
| Events | registration order, duplicate listeners, payload changes, teardown |
| UI | default/custom view, resize, accessibility, browser/mobile layout |
| Production | minification, CSP, COOP/COEP, proxies, regional endpoints, logging |

## Resources

- Main changelog: https://developers.zoom.us/changelog/
- Meeting SDK changelog: https://developers.zoom.us/changelog/meeting-sdk/
- Video SDK changelog: https://developers.zoom.us/changelog/video-sdk/
- Build Platform minimum versions: https://developers.zoom.us/docs/build/minimum-version/
- Meeting SDK minimum versions: https://developers.zoom.us/docs/meeting-sdk/minimum-version/
