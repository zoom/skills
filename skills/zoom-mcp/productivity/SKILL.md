---
name: zoom-mcp/productivity
description: Zoom AI Productivity Suite MCP guidance for creating and managing Sheets. Use for the documented Sheets MCP tools and their OAuth scopes.
triggers:
  - "zoom productivity mcp"
  - "zoom sheets mcp"
  - "ai productivity suite mcp"
  - "create zoom spreadsheet"
---

# Zoom AI Productivity Suite MCP

Use this server for AI Productivity Suite Sheets operations: create spreadsheets, read and edit
cells, and add, list, rename, or delete sheets.

## Endpoint

`https://mcp.zoom.us/mcp/productivity/streamable`

The repo MCP bundle registers it as `zoom-productivity-mcp` in [../../../.mcp.json](../../../.mcp.json).

## Authentication and setup

The documented server is user-level and lists these scopes: `sheets:read:content`,
`sheets:write:content`, `sheets:update:content`, and `sheets:delete:content`. Use a General App
with `USER_OPERATION` and request only the operations needed. See
[references/tools.md](references/tools.md) and the
[Marketplace manifest template](../../rest-api/assets/marketplace-manifest-template-for-mcp-productivity.json).

**Availability warning:** the endpoint was listed in the official catalog, but its current
unauthenticated protocol probe returned `404` with `instance not found`, and protected-resource
metadata also reported that the `mcp_productivity` instance was not found. Treat the catalog
entry as documented but currently unavailable until an authenticated `initialize` and
`tools/list` succeeds. Do not claim that tool execution has been verified.

## Official source

https://developers.zoom.us/docs/mcp/zoom-ai-productivity-suite/
