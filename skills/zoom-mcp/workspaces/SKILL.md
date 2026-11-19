---
name: zoom-mcp/workspaces
description: Zoom Workspaces MCP guidance for reservationable workspaces, locations, assets, reservations, and calendar context.
triggers:
  - "zoom workspaces mcp"
  - "workspace reservation mcp"
  - "book desk zoom mcp"
  - "zoom room reservation mcp"
---

# Zoom Workspaces MCP

Use the dedicated Zoom Workspaces MCP server for workspace reservation and management: find
workspaces, locations, and assets, then create, update, or delete reservations.

Registry identity: `io.github.zoom/zoom-workspaces`, version 1.0.0 in the 2026-11-17 snapshot.
Do not confuse it with `io.github.zoom/zoom-workspace`, the general Zoom Workspace MCP server
documented by [../SKILL.md](../SKILL.md).

## Endpoint

```text
https://mcp.zoom.us/mcp/workspace/streamable
```

The singular `workspace` URL path belongs to the plural Workspaces server. The repo MCP bundle
registers it as `zoom-workspaces-mcp` in [../../../.mcp.json](../../../.mcp.json).

## Setup

Use the parent OAuth flow in [../SKILL.md](../SKILL.md), request user scopes required by
`tools/list`, and inspect the live tool contract before making reservations.

Representative capabilities from registry metadata:

- List workspaces, accessible locations, and assets available for reservation.
- Create, update, delete, and list reservations.
- Retrieve calendar events, contacts, and meetings related to workspace plans.

Confirm date, time zone, location, asset, and recurrence with the user before writing a
reservation; cancellation and overwrite operations can disrupt other occupants.

## Official Source

https://developers.zoom.us/docs/mcp/zoom-workspaces-mcp-server/
