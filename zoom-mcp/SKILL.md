---
name: zoom-mcp
description: Connects Claude to the official Zoom MCP Server for meeting management and recording access. Creates, searches, lists, updates, and deletes Zoom meetings; lists and retrieves cloud recordings including transcripts. Use when the user asks to manage Zoom meetings, schedule calls, find past meetings, access recording transcripts, or automate Zoom workflows. Requires a Zoom OAuth Bearer token. AI Companion must be enabled for transcript access.
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
  - "meeting summary"
  - "ai companion transcript"
---

# Zoom MCP Server

Connects to Zoom's official MCP Server at `mcp-us.zoom.us` for meeting management and
recording access via the Model Context Protocol. The same server infrastructure powers
the Zoom for ChatGPT app — transcript retrieval is the primary use case.

## Quick Start

**1. Add to Claude Code:**
```bash
claude mcp add --transport http zoom-mcp \
  https://mcp-us.zoom.us/mcp/zoom/streamable \
  --header "Authorization: Bearer YOUR_ZOOM_OAUTH_TOKEN" \
  --scope user
```

**2. Verify connection:**
```
zoom-mcp:get_user_profile
```

**3. Try a search:**
```
zoom-mcp:search_meetings  query: "team sync"  startDate: "2025-01-01"  endDate: "2025-01-31"
```

For OAuth setup, scopes, and token lifecycle: [concepts/oauth-setup.md](concepts/oauth-setup.md)

---

## ⚠️ Critical Notes

**1. AI Companion required for transcripts**
Transcript files (`VTT`, `TRANSCRIPT`) only appear in `get_recording` results when
**Smart Recording** and **Meeting Summary** are enabled in your Zoom account's AI Companion
settings. This is the #1 reason transcript retrieval silently returns no transcript files.
→ Zoom web portal: Admin → Account Management → Account Settings → AI Companion

**2. Cloud recording required**
Transcripts are only generated for meetings recorded to cloud (`auto_recording: "cloud"`).
Local recordings do not produce transcript files.

**3. OAuth tokens expire after 1 hour**
Refresh with your `refresh_token` and re-run `claude mcp add` with the updated token.

**4. Past meeting IDs may differ from recording IDs**
Use `list_recordings` with a date range when `get_recording` returns 404 — recordings use
a UUID that may differ from the scheduled meeting ID.

---

## Server Endpoints

| Transport | URL |
|-----------|-----|
| Streamable HTTP (recommended) | `https://mcp-us.zoom.us/mcp/zoom/streamable` |
| SSE (fallback) | `https://mcp-us.zoom.us/mcp/zoom/sse` |

A separate Whiteboard MCP server exists at `/mcp/whiteboard/streamable` — requires a
Whiteboard instance ID and is not covered by this skill.

---

## Available Tools

Always use fully qualified tool names: `zoom-mcp:tool_name`

Full parameter schemas: [references/tools.md](references/tools.md)

### Meeting Management

| Tool | Key Parameters | Description |
|------|---------------|-------------|
| `zoom-mcp:search_meetings` | `query`, `startDate`, `endDate`, `pageSize` (max 300) | Search meetings by topic, participant, or date range |
| `zoom-mcp:list_meetings` | `type` (scheduled/live/upcoming), `pageSize` | List meetings for the authenticated user |
| `zoom-mcp:get_meeting` | `meetingId`*, `occurrenceId`, `showPreviousOccurrences` | Get details for a specific meeting |
| `zoom-mcp:create_meeting` | `topic`*, `type`*, `startTime`, `duration`, `timezone`, `agenda`, `recurrence`, `settings` | Create a new meeting |
| `zoom-mcp:update_meeting` | `meetingId`* + any create_meeting params | Update an existing meeting |
| `zoom-mcp:delete_meeting` | `meetingId`*, `occurrenceId`, `scheduleForReminder` | Delete a meeting |

**Meeting types:** `1`=instant, `2`=scheduled, `3`=recurring (no fixed time), `8`=recurring (fixed time)

### Recordings & Transcripts

| Tool | Key Parameters | Description |
|------|---------------|-------------|
| `zoom-mcp:list_recordings` | `from`*, `to`* (YYYY-MM-DD), `pageSize`, `trash` | List cloud recordings by date range |
| `zoom-mcp:get_recording` | `meetingId`* | Get recording details including all recording files |
| `zoom-mcp:delete_recording` | `meetingId`*, `action` (trash/delete) | Trash or permanently delete a recording |

**Transcript access:** `get_recording` returns a `recording_files` array. Filter for
`file_type: "VTT"` (timestamps) or `file_type: "TRANSCRIPT"` (speaker labels). Fetch
`download_url` with `Authorization: Bearer TOKEN` to retrieve the content.

### User & Utility

| Tool | Parameters | Description |
|------|------------|-------------|
| `zoom-mcp:get_user_profile` | none | Get the authenticated user's Zoom profile |
| `zoom-mcp:list_available_tools` | none | List all tools available on this server |
| `zoom-mcp:get_tool_details` | `toolName`* | Get full parameter schema for any tool by name |

\* Required parameter

---

## Key Workflows

**Retrieve a meeting transcript** (primary use case):
```
1. zoom-mcp:search_meetings  →  find meeting by topic or date range
2. zoom-mcp:get_recording    →  pass meetingId, inspect recording_files
3. Filter for file_type "TRANSCRIPT" or "VTT" → fetch download_url with Bearer token
4. Analyze: summarize, extract action items, find speaker contributions
```
Full walkthrough: [examples/transcript-retrieval.md](examples/transcript-retrieval.md)

**Create a scheduled meeting with cloud recording:**
```
zoom-mcp:create_meeting
  topic: "Weekly sync", type: 2, startTime: "2025-03-10T14:00:00"
  timezone: "America/New_York", duration: 60
  settings: { auto_recording: "cloud", waiting_room: true }
```
Full examples: [examples/meeting-lifecycle.md](examples/meeting-lifecycle.md)

**Find and reschedule a meeting:**
```
1. zoom-mcp:search_meetings  →  query: "budget review"
2. zoom-mcp:update_meeting   →  meetingId + new startTime
```
More patterns: [examples/search-and-act.md](examples/search-and-act.md)

---

## Error Reference

| Code | Meaning | Fix |
|------|---------|-----|
| `-32001 Access token required` | No or missing auth header | Re-run `mcp add` with `--header "Authorization: Bearer TOKEN"` |
| `-32001 Invalid access token` | Token expired | Refresh OAuth token; re-register |
| `-32602 instance not found` | Wrong MCP endpoint | Use `.../mcp/zoom/streamable` not `.../mcp/whiteboard/...` |
| `404` | Meeting/recording not found | Use `list_recordings` by date range to find correct UUID |
| `429` | Rate limit exceeded | Back off; limit is ~10 req/sec |

Full error reference: [references/error-codes.md](references/error-codes.md)

---

## Documentation

### Concepts
- [concepts/mcp-architecture.md](concepts/mcp-architecture.md) — MCP protocol, transports, how Claude and ChatGPT connect
- [concepts/oauth-setup.md](concepts/oauth-setup.md) — OAuth app creation, scopes, AI Companion setup, token lifecycle

### Examples
- [examples/transcript-retrieval.md](examples/transcript-retrieval.md) — **Primary use case**: find meeting → get recording → read transcript
- [examples/meeting-lifecycle.md](examples/meeting-lifecycle.md) — Create, update, and delete meetings
- [examples/search-and-act.md](examples/search-and-act.md) — Search by topic/date, then take action

### References
- [references/tools.md](references/tools.md) — Complete tool reference with all parameters
- [references/error-codes.md](references/error-codes.md) — MCP and Zoom API error codes with fixes

### Troubleshooting
- [troubleshooting/common-errors.md](troubleshooting/common-errors.md) — Auth failures, missing transcripts, connection issues

### Operations
- [RUNBOOK.md](RUNBOOK.md) — 5-minute preflight and debugging checklist

---

## Constraints

- Pagination: max 300 results per page; use `nextPageToken` for subsequent pages
- Date range windows for search and recording list: 30 days maximum
- Cloud recordings retained ~30 days by default (varies by account plan)
- Meeting passwords: 10 characters max
- Recurring meetings require type `3` or `8`

---

## Related Skills

- [zoom-rest-api](../rest-api/SKILL.md) — Direct REST API access (600+ endpoints, advanced recording management)
- [zoom-oauth](../oauth/SKILL.md) — OAuth implementation patterns (all 4 grant types)
- [zoom-webhooks](../webhooks/SKILL.md) — Receive real-time recording completion events
- [zoom-rtms](../rtms/SKILL.md) — Live audio/video/transcript streams during active meetings
