# Tool Reference — Zoom MCP Server

Tools available on the Zoom MCP server. Treat the raw server tool names as authoritative.
Some MCP clients namespace them in the UI, for example `zoom-mcp:recordings_list`.

This reference is based on the current `tools/list` output and tool execution behavior of
`https://mcp-us.zoom.us/mcp/zoom/streamable`.

Treat the live MCP protocol `tools/list` response as the authoritative source for the current
tool list and schemas.

## Current Live Tools

The current `tools/list` response exposes these Zoom MCP tools:

- `create_new_file_with_markdown`
- `get_meeting_assets`
- `search_meetings`
- `get_recording_resource`
- `recordings_list`

The server did **not** expose older inferred tool names such as `list_meetings`,
`get_meeting`, `create_meeting`, `get_user_profile`, `list_available_tools`, or
`get_tool_details` in that probe.

## Supported Scope Families Advertised by Zoom MCP

Protected-resource metadata for Zoom MCP advertised these scope families:
- `docs:write:import`
- `meeting:read:assets`
- `meeting:read:search`
- `cloud_recording:read:content`
- `cloud_recording:read:list_user_recordings`

## Documents

### `create_new_file_with_markdown`

Create a Zoom Docs document from Markdown content.

**Verified scope:** `docs:write:import`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `content` | string | **Yes** | Markdown formatted content |
| `file_name` | string | No | Name of the new document |
| `parent_id` | string | No | Parent file/folder ID; omit to place under My Docs |

Successful calls return:
- `file_id`
- `file_link`

## Meeting Discovery and Assets

### `search_meetings`

Read-only search tool for semantic meeting discovery.

**Verified scope:** `meeting:read:search`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | No | Search query keyword |
| `from` | string | No | UTC start datetime |
| `to` | string | No | UTC end datetime |
| `page_size` | integer | No | Results per page; default `50`, max `300` |
| `next_page_token` | string | No | Pagination token; expires in 15 minutes |

### `get_meeting_assets`

Read-only meeting asset hub. Retrieves meeting summary, recording, whiteboards, Zoom Docs,
and related artifacts for a specific meeting.

**Verified scope:** `meeting:read:assets`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `meetingId` | string | **Yes** | Numeric meeting number or UUID; the schema prefers UUID when available |

**Important live-schema note:**
- UUID-style values may require double encoding when they contain `/` or `//`.
- The tool description strongly prefers explicit user selection when choosing a meeting from search results.

## Recordings

### `recordings_list`

List cloud recordings for a user.

**Verified scope:** `cloud_recording:read:list_user_recordings`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `userId` | string | **Yes** | User ID; user OAuth commonly accepts `me` |
| `from` | string | No | Start date |
| `to` | string | No | End date |
| `meeting_id` | integer | No | Filter by meeting number |
| `trash` | boolean | No | Include trashed recordings |
| `trash_type` | string | No | Trash filter category |
| `mc` | string | No | Additional recording filter flag from the live schema |
| `page_size` | integer | No | Results per page; default `30`, max `300` |
| `next_page_token` | string | No | Pagination token |

### `get_recording_resource`

Retrieve recording-oriented assets for a specific meeting, including transcript-like,
summary-like, and playback-oriented resources.

**Verified scope:** `cloud_recording:read:content`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `meetingId` | string | **Yes** | Meeting UUID or recording-capable identifier |
| `types` | string | No | Resource type selector |
| `clip_num` | integer | No | Clip number / segment selector |
| `play_time` | integer | No | Playback position |
| `raw_passcode` | string | No | Plaintext recording passcode |
| `encode_passcode` | string | No | Encoded recording passcode |

**Output shape from live schema includes resource families such as:**
- transcript timelines
- summaries
- next steps
- play URLs

## Discovery Notes

- Discovery happens through MCP protocol `tools/list`, not through a dedicated Zoom utility tool.
- Re-run `tools/list` whenever you need to confirm whether the current tool list has changed.
- Do not rely on older examples that use `query`, `startDate`, `endDate`, or `pageSize`; the current live schema uses `q`, `from`, `to`, and `page_size`.
