# Tool Reference — Zoom MCP Server

Tools available on the Zoom MCP server. Treat the raw server tool names as authoritative.
Some MCP clients namespace them in the UI, for example `zoom-mcp:recordings_list`.

This reference is based on the current `tools/list` output and tool execution behavior of
`https://mcp.zoom.us/mcp/zoom/streamable`.

Treat the live MCP protocol `tools/list` response as the authoritative source for the current
tool list and schemas.

## Current Live Tools

The default Zoom MCP server currently exposes 9 tools:

- `create_new_file_with_markdown`
- `get_file_content`
- `get_meeting_assets`
- `get_recording_resource`
- `hub_create_file_from_content`
- `hub_get_file_content`
- `recordings_list`
- `search_meetings`
- `search_zoom`

The server did **not** expose older inferred tool names such as `list_meetings`,
`get_meeting`, `create_meeting`, `get_user_profile`, `list_available_tools`, or
`get_tool_details` in that probe.

## Supported Scope Families Advertised by Zoom MCP

Protected-resource metadata for Zoom MCP advertised these scope families:
- `docs:write:import`
- `docs:read:export`
- `ai_companion:read:search`
- `meeting:read:assets`
- `meeting:read:search`
- `cloud_recording:read:content`
- `cloud_recording:read:list_user_recordings`
- `hub:write:content`
- `hub:read:content`

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

### `get_file_content`

Read a Zoom Docs or My Notes file as Markdown.

**Verified scope:** `docs:read:export`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `fileId` | string | **Yes** | Unique Zoom Docs file identifier, commonly returned as `file_id` from `search_zoom` |

Successful calls return:
- file name
- full file content in Markdown format

## Hub Content

### `hub_create_file_from_content`

Create a Hub file from plain-text content.

**Verified scope:** `hub:write:content`

Supported content representations documented by Zoom:
- Docs and paper docs: Markdown
- Spreadsheets: CSV
- Maximum content size: 100 KB

Use the live `tools/list` schema for file type, filename, and destination parameters.

### `hub_get_file_content`

Retrieve Hub file content in a format supported by the file type.

**Verified scope:** `hub:read:content`

Documented export formats:
- Docs: `markdown` or `xml`
- Spreadsheets: `csv`
- Presentations: `markdown`
- Paper docs: `markdown`

Use the live `tools/list` schema for the file identifier and format parameter names.

## Meeting Discovery and Assets

### `search_meetings`

Read-only search tool for semantic meeting discovery.

**Verified scope:** `meeting:read:search`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | No | Search query keyword |
| `from` | string | No | UTC start datetime |
| `to` | string | No | UTC end datetime |
| `include_zoom_my_notes` | boolean | No | Include Zoom My Notes when the user explicitly asks for My Notes or notes |
| `page_size` | integer | No | Results per page; default `50`, max `300` |
| `next_page_token` | string | No | Pagination token; expires in 15 minutes |

**Important live-schema note:**
- The tool description requires knowing the user's current timezone before invoking the tool.
- Convert user-local time references to UTC before sending `from` and `to`.
- For "recent" or "today" queries, use the user's current day start converted to UTC.

## Cross-Zoom Search

### `search_zoom`

Read-only keyword and semantic search across Team Chat messages and Zoom Docs/My Notes.

**Verified scope:** `ai_companion:read:search`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search_entities` | array | **Yes** | One or more source descriptors with `entity_type` set to `chat` or `zoom_doc` |
| `query` | string | No | Keyword or phrase; omit to browse accessible records subject to filters and `page_size` |
| `page_size` | integer | No | Results per page; default `10`, min `1`, max `100`; no pagination cursor |

Supported `search_entities` shapes:

```json
[
  {
    "entity_type": "chat",
    "filters": {
      "channel_names": ["engineering"],
      "from": "2026-03-01T00:00:00Z",
      "to": "2026-03-06T23:59:59Z",
      "unread": false
    }
  },
  {
    "entity_type": "zoom_doc",
    "filters": {
      "doc_view": "notes",
      "from": "2026-03-01T00:00:00Z",
      "to": "2026-03-06T23:59:59Z"
    }
  }
]
```

Filter notes:
- Chat supports `channel_names`, `hsession_ids`, `from`, `to`, and `unread`.
- Zoom Docs supports `doc_view`, `from`, and `to`.
- `doc_view` values are `recent`, `my_docs`, `shared_with_me`, `starred`, and `notes`.
- `notes` refers to Zoom AI Companion notes/My Notes. Use it when the user asks for meeting notes or My Notes.
- All `from` and `to` values must be ISO 8601 UTC. Convert relative time references from the user's timezone.

Returned items are discriminated by `entity_type`:
- Chat results include channel metadata and `message_list`.
- Zoom Doc results include `title`, `file_type`, `link`, `file_id`, `create_time`, and `modify_time`.

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

List cloud recordings for the authorized user.

**Verified scope:** `cloud_recording:read:list_user_recordings`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
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
- Do not pass `userId` to `recordings_list` on the current default MCP server; it lists recordings for the authorized user and the live schema does not expose `userId`.
