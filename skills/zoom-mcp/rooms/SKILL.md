---
name: zoom-mcp/rooms
description: Zoom Rooms MCP guidance for rooms, devices, locations, tags, room settings, and calendar resources.
triggers:
  - "zoom rooms mcp"
  - "zoom room device mcp"
  - "zoom location mcp"
  - "room calendar resource mcp"
---

# Zoom Rooms MCP

Use the dedicated Zoom Rooms MCP server to list and manage rooms, devices, locations, and tags;
read room, account, and location profiles and settings; and inspect calendar resources.

Registry identity: `io.github.zoom/zoom-rooms`, version 1.0.1 in the 2026-11-17 snapshot.

## Endpoint

```text
https://mcp.zoom.us/mcp/rooms/streamable
```

The repo MCP bundle registers it as `zoom-rooms-mcp` in [../../../.mcp.json](../../../.mcp.json).

## Setup

Use the parent OAuth workflow in [../SKILL.md](../SKILL.md) and request only the scopes exposed
by the tools you need. Call `tools/list` before automation; registry metadata does not enumerate
tool names.

Representative capabilities from registry metadata:

- List rooms and devices and retrieve their details.
- Create, change, reassign, or delete locations and maintain the location hierarchy.
- Create tags and assign or unassign them on rooms.
- List calendar services and look up room calendar resources.

For device firmware, bulk provisioning, or deterministic settings changes not exposed by
`tools/list`, chain [Zoom REST API](../../rest-api/SKILL.md).

## Official Source

https://developers.zoom.us/docs/mcp/zoom-rooms-mcp-server/
