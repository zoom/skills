# MCP Architecture — Zoom MCP Server

## What is MCP?

Model Context Protocol (MCP) standardizes how AI systems connect to external tools and data
sources. Zoom exposes hosted MCP surfaces that clients can discover and call over MCP.

## Hosted Zoom MCP Surfaces

### Zoom MCP

| Transport | URL |
|-----------|-----|
| Streamable HTTP | `https://mcp.zoom.us/mcp/zoom/streamable` |

### Whiteboard MCP

| Transport | URL |
|-----------|-----|
| Streamable HTTP | `https://mcp.zoom.us/mcp/whiteboard/streamable` |

In this repo, Whiteboard MCP is handled by the child skill
[../whiteboard/SKILL.md](../whiteboard/SKILL.md).

### Team Chat MCP

| Transport | URL |
|-----------|-----|
| Streamable HTTP (recommended) | `https://mcp.zoom.us/mcp/chat/streamable` |
| Legacy alias | `https://mcp.zoom.us/mcp/team_chat/streamable` |

In this repo, Team Chat MCP is handled by the child skill
[../team-chat/SKILL.md](../team-chat/SKILL.md).

### Other Dedicated Product Servers

| Product | Streamable HTTP | Child skill |
|---------|-----------------|-------------|
| Meetings | `https://mcp.zoom.us/mcp/meeting/streamable` | [Meetings](../meetings/SKILL.md) |
| Chat | `https://mcp.zoom.us/mcp/chat/streamable` | [Team Chat](../team-chat/SKILL.md) |
| Canvas | `https://mcp.zoom.us/mcp/canvas/streamable` | [Canvas](../canvas/SKILL.md) |
| Tasks | `https://mcp.zoom.us/mcp/tasks/streamable` | [Tasks](../tasks/SKILL.md) |
| Whiteboard | `https://mcp.zoom.us/mcp/whiteboard/streamable` | [Whiteboard](../whiteboard/SKILL.md) |
| Revenue Accelerator | `https://mcp.zoom.us/mcp/revenue_accelerator/streamable` | [Revenue Accelerator](../revenue-accelerator/SKILL.md) |

The current official catalog also identifies the unified Zoom MCP Server at
`https://mcp.zoom.us/mcp/zoom/streamable`. The dedicated product servers and the unified
server use Streamable HTTP in the current catalog; do not assume SSE support.

## Discovery Model

Do not hardcode tool counts in client logic.

Use the MCP protocol `tools/list` response as the current source of truth for:
- tool names
- descriptions
- parameter schemas
- newly added or removed tools

## Current Capability Shape

The current unified Zoom MCP surface exposes 9 tools centered on:
- semantic meeting search
- cross-Zoom search over Team Chat messages and Zoom Canvas/My Notes
- meeting asset retrieval
- recording resource retrieval
- Canvas/My Notes file creation from Markdown
- Canvas/My Notes Markdown content export
- Hub file creation and multi-format content export

If the task requires deterministic meeting CRUD, use the REST API skill instead of assuming
those operations exist on the current Zoom MCP surface.

## Authentication Model

User OAuth is the primary documented path.

S2S tokens can:
- initialize against the MCP gateway
- complete `tools/list`
- open sessions in previously tested environments

Treat S2S as transport/discovery-capable unless tool execution has been separately validated
for your app and scopes. The current official server catalog documents Streamable HTTP, so do
not select SSE as a default transport.

## Protected Resource Metadata

The hosted MCP surfaces advertise supported scopes through OAuth protected-resource metadata.
Zoom MCP protected-resource metadata currently exposes:
- `docs:write:import`
- `docs:read:export`
- `ai_companion:read:search`
- `meeting:read:assets`
- `meeting:read:search`
- `cloud_recording:read:content`
- `cloud_recording:read:list_user_recordings`
- `hub:write:content`
- `hub:read:content`

Canvas MCP protected-resource metadata currently exposes:
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

Whiteboard MCP protected-resource metadata currently exposes:
- `whiteboard:write:whiteboard`
- `whiteboard:read:list_whiteboards`
- `whiteboard:read:whiteboard`
- `whiteboard:write:collaborator`
- `whiteboard:delete:collaborator`
- `whiteboard:update:collaborator`
- `whiteboard:read:list_collaborators`

Team Chat MCP protected-resource metadata currently exposes:
- `team_chat:write:user_message`
- `team_chat:update:user_message`
- `team_chat:write:contact_information`
- `team_chat:write:user_channel`
- `team_chat:update:user_channel`
- `team_chat:write:members`
- `team_chat:read:channel`
- `team_chat:update:channel_member_role`
- `team_chat:read:list_members`
- `team_chat:read:list_user_channels`
- `team_chat:read:list_contacts`
- `team_chat:read:list_user_files`
- `team_chat:read:list_user_messages`
- `team_chat:read:list_user_sessions`
- `chat_channel:read` and `chat_channel:write` metadata aliases

## Retrieval Model

`search_meetings` is not just a title filter. It is a semantic retrieval path over meeting
content, recap-linked assets, and recording-linked artifacts.

`search_zoom` is the cross-Zoom knowledge discovery path for Team Chat messages, Zoom Canvas,
and My Notes. Pair it with `get_file_content` when a returned Canvas file or My Notes file must
be read as Markdown.

Useful result families:
- recap-oriented results with AI summaries and linked assets
- recording-oriented results for post-meeting content retrieval
- chat/Canvas-oriented results for collaboration search and document inspection

When writing parsers, validate the live response shape from the server rather than relying on
older example field names.

## Feature Prerequisites

AI Companion features such as **Smart Recording** and **Meeting Summary** are feature
prerequisites for useful semantic retrieval and recap-linked content. They do not replace the
required OAuth scopes.

## Error Layering

Failures can happen at two layers:
- MCP protocol layer (`-32001`, `-32602`, `-32603`)
- underlying Zoom API-style permission/resource failures surfaced through the MCP response

See [../references/error-codes.md](../references/error-codes.md).
