# RTMS Skill Changelog

## 2026-10-15

### Changed

- Deprecated unified media socket mode, `server_urls.all`, and `media_type: 32`.
- Updated guidance to open one data WebSocket per selected concrete media type: audio (`1`), video (`2`), deskshare (`4`), transcript (`8`), or chat (`16`).
- Clarified that SDK or library selection masks must be expanded into separate wire-level data handshakes.
- Updated connection, lifecycle, manual WebSocket, bot, troubleshooting, runbook, and environment configuration examples for split media sockets.
