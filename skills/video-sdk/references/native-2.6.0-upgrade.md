# Native Video SDK 2.6.0 Upgrade

Public changelog baseline verified on 2026-07-10 for Android, iOS, Linux, macOS, and Windows.
Electron also has a `2.6.0` release, but this repository does not currently expose a dedicated
Electron Video SDK child skill.

## Shared Additions

- Add two-step `prepareJoin` / `commitJoin` pre-connection flows.
- Add I420 Limited and I420 Full external-video formats.
- Add machine-answer detection for PSTN call-outs.
- Add user failover status tracking.
- Add detailed audio QoS and jitter-buffer statistics.
- Expand command-channel payloads to 1,024 bytes.
- Add subsession status/leave callback coverage.

## Breaking-Change Checklist

- Recompile against the `2.6.0` headers; do not reuse `2.5.10` binaries or copied declarations.
- Update renamed audio-configuration APIs.
- Configure external video format in session context where required.
- Remove obsolete `format` arguments from `sendVideoFrame` where the platform changed it.
- Re-run callback/delegate completeness checks for new subsession events.
- Validate external video color range end-to-end; Limited and Full are not interchangeable.

## Platform Notes

| Platform | Additional migration notes |
|----------|----------------------------|
| Windows | Requires VC++ 2022 redistributable; share pause/resume now supports preprocess share. |
| iOS | Uses a new singleton accessor and requires `util.framework` plus `zContext.framework`. |
| Linux | Removes annotation interfaces and the XMPP command channel. |
| macOS | Corrects statistic property names, adds sharing-window state notifications and second-camera sharing, and removes `xmpp_framework.framework`. |
| Android | Includes the shared API renames, external-source changes, pre-connect flow, QoS, failover, and subsession additions. |

## Source Policy

The public changelog describes release-level behavior. The downloaded platform package headers,
frameworks, samples, and bundled API skill are authoritative for exact names and signatures.
Existing `2.5.10` references in this repository remain useful historical inventories but must not
be treated as `2.6.0` API definitions.

Official changelog: https://developers.zoom.us/changelog/video-sdk/
