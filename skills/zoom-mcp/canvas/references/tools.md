# Tools - Zoom Canvas MCP

Current catalog for `https://mcp.zoom.us/mcp/canvas/streamable`.
Run `tools/list` before relying on this inventory because Zoom can add or change tools.

| Tool | Required scope |
|------|----------------|
| `canvas_add_file_collaborators` | `docs:write:collaborator` |
| `canvas_delete_block` | `docs:delete:content` |
| `canvas_delete_file` | `docs:delete:file` |
| `canvas_get_file_general_access_setting` | `docs:read:general_access` |
| `canvas_get_file_metadata` | `docs:read:file` |
| `canvas_get_spec` | `docs:read:export` |
| `canvas_insert_block` | `docs:write:content` |
| `canvas_list_all_file_children` | `docs:read:list_children` |
| `canvas_list_file_collaborators` | `docs:read:list_file_collaborators` |
| `canvas_modify_file_collaborator_role` | `docs:update:collaborator` |
| `canvas_modify_file_general_access_setting` | `docs:update:general_access` |
| `canvas_modify_file_metadata` | `docs:update:file` |
| `canvas_remove_file_collaborator` | `docs:delete:collaborator` |
| `canvas_replace_range_of_blocks` | `docs:update:content` |
| `canvas_transfer_file_ownership` | `docs:update:file_owner` |
| `canvas_update_block` | `docs:update:content` |
| `create_file_with_content` | `docs:write:import` |
| `get_file_content` | `docs:read:export` |

## Routing

- Use this child skill for current Canvas MCP operations.
- Use [../SKILL.md](../SKILL.md) for server selection and OAuth chaining.
- Use the REST Docs/Canvas references for deterministic API workflows.
