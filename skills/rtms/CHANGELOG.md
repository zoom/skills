# RTMS Skill Changelog

## 2026-11-27

### Changed

- Documented by-design RTMS stream deduplication: a host and participant under the same account starting the same RTMS Marketplace app in one meeting produce a single `meeting.rtms_started` event.
- Documented that when the original host configured auto-start, the host's user ID is always present in the webhook payload (`object.host_id`); sessions should be keyed on `rtms_stream_id`, not user ID.
- Source: Zoom-confirmed behavior supplied by the requester; not yet in a provenance-stamped docs snapshot.

## 2026-10-15

### Changed

- Deprecated unified media socket mode, `server_urls.all`, and `media_type: 32`.
- Updated guidance to open one data WebSocket per selected concrete media type: audio (`1`), video (`2`), deskshare (`4`), transcript (`8`), or chat (`16`).
- Clarified that SDK or library selection masks must be expanded into separate wire-level data handshakes.
- Updated connection, lifecycle, manual WebSocket, bot, troubleshooting, runbook, and environment configuration examples for split media sockets.
