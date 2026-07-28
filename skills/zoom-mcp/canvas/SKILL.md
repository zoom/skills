---
name: zoom-mcp/canvas
description: |
  Zoom Canvas MCP server guidance for Canvas file and block operations, collaborators,
  sharing settings, ownership, Markdown import, and content export. Use for the current
  Canvas MCP endpoint and OAuth scopes; use the parent Zoom MCP skill for server selection.
triggers:
  - "zoom canvas mcp"
  - "canvas mcp server"
  - "zoom canvas via mcp"
  - "create canvas file mcp"
  - "update canvas block mcp"
  - "canvas collaborator mcp"
  - "zoom docs mcp"
  - "docs mcp server"
---

# Zoom Canvas MCP

Use the dedicated Canvas MCP server for agent-driven Canvas file, block, collaborator,
sharing, ownership, import, and content export workflows. The current official product name
is **Zoom Canvas MCP Server**; do not select the old Docs MCP endpoint.

## Endpoint

```text
https://mcp.zoom.us/mcp/canvas/streamable
```

Use `tools/list` as the runtime source of truth for tool names and schemas. The current
official catalog exposes 18 tools. See [references/tools.md](references/tools.md).

## Required Scopes

Add only the scopes required by the requested operations:

- `docs:read:export`
- `docs:read:file`
- `docs:read:general_access`
- `docs:read:list_children`
- `docs:read:list_file_collaborators`
- `docs:write:import`
- `docs:write:content`
- `docs:write:collaborator`
- `docs:update:content`
- `docs:update:file`
- `docs:update:file_owner`
- `docs:update:general_access`
- `docs:update:collaborator`
- `docs:delete:content`
- `docs:delete:file`
- `docs:delete:collaborator`

These are user-level OAuth scopes for the current hosted Canvas server. Do not substitute
S2S or Meeting SDK credentials.

## Chaining

- Marketplace app creation: [Canvas MCP template](../../rest-api/assets/marketplace-apps/marketplace-manifest-template-for-mcp-canvas.json)
  via [Marketplace app management](../../rest-api/references/marketplace-apps.md)
- Token acquisition: create the app first, then use [OAuth and PKCE](../../oauth/SKILL.md) to
  authorize the user and mint the bearer token supplied to this endpoint
- Parent MCP routing: [../SKILL.md](../SKILL.md)
- Canvas REST API fallback: [../../rest-api/references/zoom-docs.md](../../rest-api/references/zoom-docs.md)

## Safety Rules

- Confirm destructive operations before deleting files, blocks, or collaborators.
- Preserve file and block IDs returned by discovery; do not guess identifiers.
- Use REST APIs for deterministic bulk jobs, custom retries, and audit-heavy workflows.

## Official Source

https://developers.zoom.us/docs/mcp/zoom-canvas-mcp-server/
