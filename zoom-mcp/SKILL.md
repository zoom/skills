---
name: zoom-mcp
description: Connects Claude to the official Zoom MCP Server for meeting management and recording access. Creates, searches, lists, updates, and deletes Zoom meetings; lists and retrieves cloud recordings including transcripts. Use when the user asks to manage Zoom meetings, schedule calls, find past meetings, access recording transcripts, or automate Zoom workflows. Requires a Zoom OAuth Bearer token.
triggers:
  - "zoom meeting"
  - "search meetings"
  - "list meetings"
  - "create meeting"
  - "schedule zoom"
  - "find meeting"
  - "update meeting"
  - "delete meeting"
  - "cancel meeting"
  - "zoom transcript"
  - "meeting transcript"
  - "zoom recording"
  - "cloud recording"
  - "zoom mcp"
---

# Zoom MCP Server

Connects to Zoom's official MCP Server at `mcp-us.zoom.us` for meeting management and
recording access via the Model Context Protocol.

## Server Endpoints

| Transport | URL |
|-----------|-----|
| Streamable HTTP (recommended) | `https://mcp-us.zoom.us/mcp/zoom/streamable` |
| SSE | `https://mcp-us.zoom.us/mcp/zoom/sse` |

A separate Zoom Whiteboard MCP server exists at `https://mcp-us.zoom.us/mcp/whiteboard/streamable`
and `https://mcp-us.zoom.us/mcp/whiteboard/sse` — requires a Whiteboard instance ID; not
covered by this skill.

## Setup

Requires a Zoom OAuth Bearer token. Add to Claude Code:

```bash
claude mcp add --transport http zoom-mcp \
  https://mcp-us.zoom.us/mcp/zoom/streamable \
  --header "Authorization: Bearer YOUR_ZOOM_OAUTH_TOKEN" \
  --scope user
```

**OAuth token:** Create a User-managed OAuth app in the [Zoom Marketplace](https://marketplace.zoom.us)
with `meeting:read`, `meeting:write`, and `recording:read` scopes.

## Available Tools

Always use fully qualified tool names: `zoom-mcp:tool_name`

### Meeting Management

| Tool | Key Parameters | Description |
|------|---------------|-------------|
| `zoom-mcp:search_meetings` | `query`, `startDate`, `endDate`, `pageSize` (max 300) | Search meetings by topic, participant, or date range |
| `zoom-mcp:list_meetings` | `type` (scheduled/live/upcoming), `pageSize` | List meetings for the authenticated user |
| `zoom-mcp:get_meeting` | `meetingId` *, `occurrenceId`, `showPreviousOccurrences` | Get details for a specific meeting |
| `zoom-mcp:create_meeting` | `topic` *, `type` *, `startTime`, `duration`, `timezone`, `agenda`, `recurrence`, `settings` | Create a new meeting |
| `zoom-mcp:update_meeting` | `meetingId` * + any create_meeting params | Update an existing meeting |
| `zoom-mcp:delete_meeting` | `meetingId` *, `occurrenceId`, `scheduleForReminder` | Delete a meeting |

**Meeting types:** `1`=instant, `2`=scheduled, `3`=recurring (no fixed time), `8`=recurring (fixed time)

**Recurrence object** (for type 3 or 8): `type` (1=daily, 2=weekly, 3=monthly), `repeat_interval`,
`weekly_days`, `end_times` or `end_date_time`

**Settings object:** `host_video`, `participant_video`, `join_before_host`, `mute_upon_entry`,
`waiting_room`, `auto_recording` (local/cloud/none)

### Recordings & Transcripts

| Tool | Key Parameters | Description |
|------|---------------|-------------|
| `zoom-mcp:list_recordings` | `from`, `to` (YYYY-MM-DD), `pageSize`, `trash` | List cloud recordings by date range |
| `zoom-mcp:get_recording` | `meetingId` * | Get recording details including all recording files |
| `zoom-mcp:delete_recording` | `meetingId` *, `action` (trash/delete) | Trash or permanently delete a recording |

**Transcript access:** `zoom-mcp:get_recording` returns a `recording_files` array. Filter for
`file_type: "VTT"` or `file_type: "TRANSCRIPT"` and fetch the `download_url` to retrieve
transcript content. VTT includes timestamps; TRANSCRIPT is structured JSON with speaker labels.

### User & Utility

| Tool | Parameters | Description |
|------|------------|-------------|
| `zoom-mcp:get_user_profile` | none | Get the authenticated user's Zoom profile |
| `zoom-mcp:list_available_tools` | none | List all tools available on this server |
| `zoom-mcp:get_tool_details` | `toolName` * | Get full parameter schema for any tool by name |

\* Required parameter

## Key Workflows

**Retrieve a meeting transcript:**
```
1. zoom-mcp:search_meetings  →  find meeting by topic or date
2. zoom-mcp:get_recording    →  pass meetingId, inspect recording_files
3. Find file_type "VTT" or "TRANSCRIPT" in recording_files
4. Fetch the download_url to read transcript text
```
Note: transcripts require cloud recording and audio transcription enabled on the Zoom account.

**Create a recurring weekly meeting:**
```
zoom-mcp:create_meeting with:
  type: 8, startTime: "2025-03-03T09:00:00", timezone: "America/New_York"
  recurrence: { type: 2, repeat_interval: 1, weekly_days: "2" }
  settings: { auto_recording: "cloud", waiting_room: true }
```

**Find and reschedule a meeting:**
```
1. zoom-mcp:search_meetings  →  query: "budget review"
2. zoom-mcp:update_meeting   →  meetingId + new startTime
```

## Error Handling

| Code | Meaning | Action |
|------|---------|--------|
| `-32001` | Access token required or invalid | Refresh OAuth token; re-run `mcp add` with updated `--header` |
| `400` | Invalid parameters | Check date formats (YYYY-MM-DD / ISO 8601); verify required fields |
| `404` | Meeting or recording not found | Confirm ID is correct; check if already deleted |
| `429` | Rate limit exceeded | Back off and retry; default limit is ~10 requests/second |

## Constraints

- Pagination: max 300 results per page; use `nextPageToken` for subsequent pages
- Date range windows for search and recording list: 30 days max
- Meeting passwords: 10 characters max
- Recurring meetings require `type` 3 or 8
- Cloud recordings retained ~30 days by default (varies by account plan)
- Transcripts only available when the host's account has cloud recording and
  audio transcription enabled in Zoom settings
