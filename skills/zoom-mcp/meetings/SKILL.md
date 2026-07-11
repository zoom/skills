---
name: zoom-mcp/meetings
description: |
  Zoom Meetings MCP server guidance for semantic meeting search, meeting assets, cloud
  recording listings, and recording-resource retrieval. Use for the dedicated Meetings MCP
  endpoint and its four read-oriented tools rather than meeting CRUD or generic Zoom search.
triggers:
  - "zoom meetings mcp"
  - "meeting mcp server"
  - "search meetings mcp"
  - "meeting assets mcp"
  - "recording resource mcp"
---

# Zoom Meetings MCP

Use the dedicated Meetings MCP server for agent-driven meeting search and asset retrieval.
Use [../../rest-api/SKILL.md](../../rest-api/SKILL.md) for deterministic meeting CRUD.

## Endpoint

`https://mcp.zoom.us/mcp/meeting/streamable`

The repo MCP bundle registers this endpoint as `zoom-meetings-mcp` in
[../../../.mcp.json](../../../.mcp.json).

## Tools and Scopes

| Tool | Scope |
|------|-------|
| `search_meetings` | `meeting:read:search` |
| `get_meeting_assets` | `meeting:read:assets` |
| `recordings_list` | `cloud_recording:read:list_user_recordings` |
| `get_recording_resource` | `cloud_recording:read:content` |

See [references/tools.md](references/tools.md) for selection and prerequisites.

## Chaining

- Marketplace app creation: [Meetings MCP template](../../rest-api/assets/marketplace-apps/zoom-mcp-meetings.json)
  via [Marketplace app management](../../rest-api/references/marketplace-apps.md)
- Token acquisition: create the app first, then use [OAuth and PKCE](../../oauth/SKILL.md) to
  authorize the user and mint the bearer token supplied to this MCP endpoint
- Parent MCP routing: [../SKILL.md](../SKILL.md)
- Deterministic meeting APIs: [../../rest-api/SKILL.md](../../rest-api/SKILL.md)
- Recording pipelines: [../../general/use-cases/recording-download-pipeline.md](../../general/use-cases/recording-download-pipeline.md)

## Official Source

https://developers.zoom.us/docs/mcp/zoom-meetings-mcp-server/
