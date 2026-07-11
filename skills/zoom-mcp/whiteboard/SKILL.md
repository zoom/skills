---
name: zoom-mcp/whiteboard
description: |
  Zoom Whiteboard MCP server guidance. Use for Whiteboard MCP auth, endpoints, ID mapping,
  and tool workflows for creating, reading, listing, and sharing Whiteboards. Prefer this
  child skill when the request is specifically about Whiteboard MCP rather than general Zoom MCP.
triggers:
  - "whiteboard mcp"
  - "zoom whiteboard mcp"
  - "zoom mcp whiteboard"
  - "zoom whiteboard tools"
  - "list zoom whiteboards"
  - "get zoom whiteboard"
  - "zoom whiteboard id"
  - "zoom wb/db"
---

# Zoom MCP Whiteboard

Dedicated guidance for Zoom's Whiteboard MCP server.

## Endpoints

| Transport | URL |
|-----------|-----|
| Streamable HTTP (recommended) | `https://mcp.zoom.us/mcp/whiteboard/streamable` |
| SSE (fallback) | `https://mcp.zoom.us/mcp/whiteboard/sse` |

## Authentication

- Start app registration from the
  [Whiteboard MCP template](../../rest-api/assets/marketplace-apps/zoom-mcp-whiteboard.json).
- **User OAuth with Whiteboard scopes** is the verified working path for `list_whiteboards`
  and `get_a_whiteboard`.
- **S2S OAuth** can reach the Whiteboard MCP gateway and complete `tools/list`, but tool
  execution must be validated separately for your app and Whiteboard scopes.
- Practical rule: start with **user OAuth** for Whiteboard MCP unless you have already
  proven your S2S app can mint and execute with the required Whiteboard scopes.

Reference: [references/authentication-and-identifiers.md](references/authentication-and-identifiers.md)

## Required Scopes

Whiteboard MCP read scopes:
- `whiteboard:read:list_whiteboards`
- `whiteboard:read:whiteboard`

Write-capable Whiteboard metadata advertised by the gateway:
- `whiteboard:write:whiteboard`
- `whiteboard:write:collaborator`
- `whiteboard:delete:collaborator`
- `whiteboard:update:collaborator`
- `whiteboard:read:list_collaborators`

## Whiteboard ID Mapping

For Whiteboard MCP, use the identifier from the URL segment after `/wb/db/`, not the numeric
segment after `/p/`.

Example:

```text
https://us05whiteboard.zoom.us/wb/db/6iktP8hJT3e5qaCuwFuAGg/p/180968285929472
                                     ^^^^^^^^^^^^^^^^^^^^^^   ^^^^^^^^^^^^^^^
                                     whiteboard_id            page/subresource id
```

## Available Tools

The current Whiteboard MCP tool surface is:

- `create_a_whiteboard_for_brainstorming`
- `list_whiteboards`
- `create_a_whiteboard`
- `get_a_whiteboard`
- `create_a_whiteboard_by_script`
- `create_a_whiteboard_for_meeting_summary`
- `create_a_whiteboard_for_strategy_analysis`
- `add_a_whiteboard_collaborator`
- `delete_a_whiteboard_collaborator`
- `get_a_whiteboard_collaborator`
- `update_a_whiteboard_collaborator`

Some MCP clients namespace server tools in the UI. Treat the raw tool names above as
authoritative.

Older catalogs included `update_a_whiteboard_metadata`; it is not in the current official tool
catalog. Re-run `tools/list` before attempting metadata updates.

Reference: [references/tools.md](references/tools.md)

## Read Workflow

1. Use a user OAuth token with the Whiteboard read scopes.
2. Call `list_whiteboards` to discover accessible whiteboards and confirm the correct `whiteboard_id`.
3. Call `get_a_whiteboard` with that `whiteboard_id`.

## Chaining

- Marketplace app creation: [Whiteboard MCP template](../../rest-api/assets/marketplace-apps/zoom-mcp-whiteboard.json)
  via [Marketplace app management](../../rest-api/references/marketplace-apps.md)
- Token acquisition: create the app first, then use
  [OAuth guidance](../concepts/oauth-setup.md) to authorize the user and mint the bearer token
  supplied to this MCP endpoint
- Parent MCP skill: [../SKILL.md](../SKILL.md)
- General routing: [../../general/SKILL.md](../../general/SKILL.md)

## References

- [references/authentication-and-identifiers.md](references/authentication-and-identifiers.md) - Auth behavior and Whiteboard ID mapping.
- [references/tools.md](references/tools.md) - Whiteboard MCP tool catalog.
