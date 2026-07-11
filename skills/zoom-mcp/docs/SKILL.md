---
name: zoom-mcp/docs
description: |
  Zoom Docs MCP server guidance for creating Docs from Markdown and retrieving file content.
  Use for the dedicated Docs MCP endpoint, Docs OAuth scopes, and deciding between MCP Docs
  tools and deterministic Zoom Docs REST block/content APIs.
triggers:
  - "zoom docs mcp"
  - "create zoom doc mcp"
  - "get zoom doc content mcp"
  - "create_file_with_content"
---

# Zoom Docs MCP

Use the dedicated Docs MCP server for agent-driven Markdown document creation and retrieval.

## Endpoint

Runtime-tested endpoint:

`https://mcp.zoom.us/mcp/docs/streamable`

The 2026-07-10 documentation page advertised `/mcp/canvas/streamable`, but that path returned
`instance not found` while `/mcp/docs/streamable` returned the expected OAuth challenge. Treat
`tools/list` and the active production endpoint as authoritative until the documentation and
runtime converge.

## Tools and Scopes

| Tool | Scope |
|------|-------|
| `create_file_with_content` | `docs:write:import` |
| `get_file_content` | `docs:read:export` |

The current REST OpenAPI inventory also exposes block-level insert/update/delete, content,
ownership, archive, and import operations. Use [Zoom Docs REST](../../rest-api/references/zoom-docs.md)
when deterministic editing or block-level control is required.

## Chaining

- Marketplace app creation: [Docs MCP template](../../rest-api/assets/marketplace-apps/zoom-mcp-docs.json)
  via [Marketplace app management](../../rest-api/references/marketplace-apps.md)
- Token acquisition: create the app first, then use [OAuth and PKCE](../../oauth/SKILL.md) to
  authorize the user and mint the bearer token supplied to this MCP endpoint
- Parent MCP routing: [../SKILL.md](../SKILL.md)
- Zoom Docs REST inventory: [../../rest-api/references/zoom-docs.md](../../rest-api/references/zoom-docs.md)

## Official Source

https://developers.zoom.us/docs/mcp/zoom-docs-mcp-server/
