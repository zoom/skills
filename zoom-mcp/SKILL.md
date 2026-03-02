---
name: zoom-mcp
description: Connects Claude to the official Zoom MCP Server for meeting management, recording access, and Zoom Doc creation. Uses AI Companion's Agentic Retrieval API to search meeting content semantically — returns inline AI summaries, Zoom Doc URLs, recording references, and whiteboards. Creates, searches, lists, updates, and deletes Zoom meetings; retrieves cloud recordings; creates Zoom Docs. Use when the user asks to manage Zoom meetings, find past meetings by content, access AI meeting summaries, retrieve transcripts, create meeting notes as a Zoom Doc, or automate Zoom workflows. Requires a Zoom OAuth Bearer token and AI Companion enabled.
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
  - "agentic retrieval"
  - "zoom doc"
  - "create doc"
  - "meeting notes"
---

# Zoom MCP Server

Connects to Zoom's official MCP Server at `mcp-us.zoom.us`. The same server powers
the Zoom for ChatGPT app. Core capabilities:

- **Agentic Retrieval search** — semantic content search across your meetings via AI Companion;
  results include inline AI summaries, Zoom Doc URLs, recording references, and whiteboards
- **Meeting management** — create, list, search, update, delete meetings
- **Recordings** — list and retrieve cloud recordings with transcript files
- **Zoom Docs** — create native Zoom documents (meeting notes, action items)

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

**3. Search meeting content:**
```
zoom-mcp:search_meetings  query: "action items from Q4 planning"
```

For OAuth setup, scopes, and token lifecycle: [concepts/oauth-setup.md](concepts/oauth-setup.md)

---

## ⚠️ Critical Notes

**1. AI Companion required for search and transcripts**
The Agentic Retrieval search and transcript access both depend on AI Companion. Enable
**Smart Recording** and **Meeting Summary** in Zoom web portal:
Admin → Account Management → Account Settings → AI Companion.
Without this, search returns no content results and recording files contain no transcripts.

**2. Cloud recording required for transcripts**
Transcript files only exist for meetings recorded to cloud (`auto_recording: "cloud"`).
Local recordings do not generate transcripts.

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

## Search & Retrieval

`search_meetings` uses AI Companion's **Agentic Retrieval API** — not a simple metadata
filter. It searches the actual content of your meetings: what was said, documents shared,
AI summaries generated.

**Two result types are returned:**

**Recap meeting** — for meetings with AI Companion data:
- Inline AI summary (text returned directly — no separate fetch needed)
- Attached Zoom Docs (URLs included)
- Recording reference
- Whiteboards

**View recording** — full recording details for meetings with cloud recordings

> **[CONTRIBUTOR NEEDED]** Exact field names for both result types. What are the JSON
> property names for: the inline AI summary text, the docs array, the recording reference,
> and the whiteboards in a Recap meeting result? What fields does a View recording result
> contain? Source: Zoom MCP Server engineering team or API schema.

See [examples/transcript-retrieval.md](examples/transcript-retrieval.md) for the full workflow.

---

## Available Tools

Always use fully qualified tool names: `zoom-mcp:tool_name`

Full parameter schemas: [references/tools.md](references/tools.md)

### Meeting Management

| Tool | Key Parameters | Description |
|------|---------------|-------------|
| `zoom-mcp:search_meetings` | `query`, `startDate`, `endDate`, `pageSize` (max 300) | Semantic content search via AI Companion Agentic Retrieval; returns Recap and View recording results |
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
| `zoom-mcp:get_recording` | `meetingId`* | Get recording details including all recording files (VTT, TRANSCRIPT, MP4) |
| `zoom-mcp:delete_recording` | `meetingId`*, `action` (trash/delete) | Trash or permanently delete a recording |

**Transcript access:** `get_recording` returns a `recording_files` array. Filter for
`file_type: "VTT"` (with timestamps) or `file_type: "TRANSCRIPT"` (with speaker labels).
Fetch `download_url` with `Authorization: Bearer TOKEN` to retrieve content.

### Zoom Docs

> **[CONTRIBUTOR NEEDED]** Verify the exact tool name for Zoom Doc creation. The tool name
> below is a placeholder. Source: `zoom-mcp:list_available_tools` or engineering team.

| Tool | Key Parameters | Description |
|------|---------------|-------------|
| `zoom-mcp:create_zoom_doc` *(verify name)* | `[CONTRIBUTOR NEEDED]` | Creates a Zoom Doc (native Zoom document, like a Notion page). Returns the URL of the created doc. |

### User & Utility

| Tool | Parameters | Description |
|------|------------|-------------|
| `zoom-mcp:get_user_profile` | none | Get the authenticated user's Zoom profile |
| `zoom-mcp:list_available_tools` | none | List all tools available on this server |
| `zoom-mcp:get_tool_details` | `toolName`* | Get full parameter schema for any tool by name |

\* Required parameter

---

## Key Workflows

**Search meeting content and get AI summary** (primary use case):
```
zoom-mcp:search_meetings  query: "Q4 planning discussion"
→ Recap meeting result includes inline AI summary, any attached Zoom Docs, recording reference
→ For most use cases, the summary is sufficient — no separate recording fetch needed
```
Full walkthrough: [examples/transcript-retrieval.md](examples/transcript-retrieval.md)

**Create a Zoom Doc from meeting content:**
```
1. zoom-mcp:search_meetings  →  find the meeting, get AI summary from Recap result
2. zoom-mcp:create_zoom_doc  →  pass content (title, action items, notes)
→ Returns URL of the new Zoom Doc
```
Full example: [examples/create-zoom-doc.md](examples/create-zoom-doc.md)

**Create a scheduled meeting with cloud recording:**
```
zoom-mcp:create_meeting
  topic: "Weekly sync", type: 2, startTime: "2025-03-10T14:00:00"
  timezone: "America/New_York", duration: 60
  settings: { auto_recording: "cloud", waiting_room: true }
```
Full examples: [examples/meeting-lifecycle.md](examples/meeting-lifecycle.md)

**Access full recording / raw transcript:**
```
1. zoom-mcp:search_meetings or list_recordings  →  find the recording
2. zoom-mcp:get_recording    →  inspect recording_files for VTT or TRANSCRIPT
3. Fetch download_url with Authorization: Bearer TOKEN
```
Use this when you need timestamped VTT, speaker-labeled JSON, or the raw video file —
beyond what the Recap AI summary provides.

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
- [concepts/mcp-architecture.md](concepts/mcp-architecture.md) — MCP protocol, Agentic Retrieval API, transports, how Claude and ChatGPT connect
- [concepts/oauth-setup.md](concepts/oauth-setup.md) — OAuth app creation, scopes, AI Companion setup, token lifecycle

### Examples
- [examples/transcript-retrieval.md](examples/transcript-retrieval.md) — **Primary use case**: Agentic Retrieval search → inline AI summary → optional raw recording
- [examples/create-zoom-doc.md](examples/create-zoom-doc.md) — Create a Zoom Doc from meeting content
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
