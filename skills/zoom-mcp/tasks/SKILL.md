---
name: zoom-mcp/tasks
description: |
  Zoom Tasks MCP server guidance for creating, reading, updating, trashing, commenting on,
  assigning, and collaborating on tasks and task steps. Use for agent-driven Tasks workflows
  over the dedicated Zoom Tasks MCP endpoint.
triggers:
  - "zoom tasks mcp"
  - "create task mcp"
  - "task assignee mcp"
  - "task steps mcp"
---

# Zoom Tasks MCP

Use this write-capable MCP surface for agent-driven task management.

## Endpoint

`https://mcp.zoom.us/mcp/tasks/streamable`

The repo MCP bundle registers it as `zoom-tasks-mcp` in
[../../../.mcp.json](../../../.mcp.json).

## Safety

Confirm the target task, assignee, collaborator, comment, or step before write/delete calls.
Do not guess task IDs or user identities. Prefer the REST Tasks API for bulk imports, durable
retry logic, and audit-heavy automation.

## Tools

See [references/tools.md](references/tools.md) for the current 20-tool catalog and scopes.

## Chaining

- Marketplace app creation: [Tasks MCP template](../../rest-api/assets/marketplace-apps/zoom-mcp-tasks.json)
  via [Marketplace app management](../../rest-api/references/marketplace-apps.md)
- Token acquisition: create the app first, then use [OAuth and PKCE](../../oauth/SKILL.md) to
  authorize the user and mint the bearer token supplied to this MCP endpoint
- Parent MCP routing: [../SKILL.md](../SKILL.md)
- Deterministic Tasks APIs: [../../rest-api/references/tasks.md](../../rest-api/references/tasks.md)

## Official Source

https://developers.zoom.us/docs/mcp/zoom-tasks-mcp-server/
