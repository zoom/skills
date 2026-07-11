# Tools — Whiteboard MCP

Current `tools/list` result from `https://mcp.zoom.us/mcp/whiteboard/streamable`.

## Tool Catalog

| Tool | Purpose | Guide coverage |
|------|---------|----------------|
| `create_a_whiteboard_for_brainstorming` | Create a whiteboard from brainstorm inputs such as problems, ideas, and next steps | Available on the Whiteboard MCP surface; validate request schema before use |
| `list_whiteboards` | List accessible whiteboards for the current user or admin context | Covered by the read workflow in this guide |
| `create_a_whiteboard` | Create a whiteboard | Available on the Whiteboard MCP surface; validate request schema before use |
| `get_a_whiteboard` | Retrieve a whiteboard by `whiteboard_id` | Covered by the read workflow in this guide |
| `create_a_whiteboard_by_script` | Create a whiteboard using scripted content or structured input | Available on the Whiteboard MCP surface; validate request schema before use |
| `create_a_whiteboard_for_meeting_summary` | Generate a whiteboard from meeting-summary style input | Available on the Whiteboard MCP surface; validate request schema before use |
| `create_a_whiteboard_for_strategy_analysis` | Generate a whiteboard for strategy-analysis content | Available on the Whiteboard MCP surface; validate request schema before use |
| `add_a_whiteboard_collaborator` | Add collaborators to a whiteboard | Confirm whiteboard and collaborator identities before writing |
| `delete_a_whiteboard_collaborator` | Remove a collaborator from a whiteboard | Destructive; require explicit confirmation |
| `get_a_whiteboard_collaborator` | List collaborators on a whiteboard | Read-only collaborator discovery |
| `update_a_whiteboard_collaborator` | Update a collaborator's whiteboard access | Confirm target identity and requested role |

`update_a_whiteboard_metadata` appeared in an older catalog but is absent from the current
official server documentation. Treat `tools/list` as authoritative before using it.

## Supported Scope Families Advertised by Whiteboard MCP

Protected-resource metadata for Whiteboard MCP advertised:
- `whiteboard:write:whiteboard`
- `whiteboard:read:list_whiteboards`
- `whiteboard:read:whiteboard`
- `whiteboard:write:collaborator`
- `whiteboard:delete:collaborator`
- `whiteboard:update:collaborator`
- `whiteboard:read:list_collaborators`
