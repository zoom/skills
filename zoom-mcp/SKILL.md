---
name: zoom-mcp
description: |
  Official Zoom MCP Server guidance for AI-agent access to semantic meeting search,
  meeting assets, recording resources, and Zoom Docs creation over MCP. Use when the
  request is about Zoom tools/list or tools/call against Zoom's hosted MCP endpoints,
  AI Companion retrieval, recording-content access, or Zoom Docs creation via MCP.
  Route Whiteboard-specific MCP requests to zoom-mcp/whiteboard.
triggers:
  - "zoom mcp"
  - "zoom mcp server"
  - "zoom mcp tools"
  - "zoom tools/list"
  - "zoom tools/call"
  - "ai companion transcript"
  - "agentic retrieval"
  - "zoom semantic meeting search"
  - "zoom search meetings by content"
  - "zoom meeting assets mcp"
  - "zoom recording resource mcp"
  - "zoom docs via mcp"
  - "zoom transcript via mcp"
  - "meeting transcript via mcp"
---

# Zoom MCP Server

Zoom hosts an MCP server at `mcp-us.zoom.us` for AI-agent access to:

- semantic meeting search
- meeting-linked asset retrieval
- recording resource retrieval
- Zoom Docs creation from Markdown

Current tool names from the Zoom MCP server:

- `create_new_file_with_markdown`
- `get_meeting_assets`
- `search_meetings`
- `get_recording_resource`
- `recordings_list`

Some MCP clients namespace server tools in the UI, for example `zoom-mcp:recordings_list`.
Treat the raw tool names above as authoritative.

Whiteboard-specific MCP work is split into the child skill
[whiteboard/SKILL.md](whiteboard/SKILL.md).

## Quick Start

**1. Add to an MCP client (Claude Code example):**
```bash
claude mcp add --transport http zoom-mcp \
  https://mcp-us.zoom.us/mcp/zoom/streamable \
  --header "Authorization: Bearer YOUR_ZOOM_OAUTH_TOKEN" \
  --scope user
```

**2. Verify discovery:**
- Confirm the client can see `recordings_list`, `search_meetings`, `get_meeting_assets`,
  `get_recording_resource`, and `create_new_file_with_markdown`.
- If the client exposes raw protocol inspection, `tools/list` is the authoritative discovery source.
- The current catalog is documented in [references/tools.md](references/tools.md).

**3. Run the first useful call:**
```text
recordings_list
  userId: "me"
  from: "2026-03-01"
  to: "2026-03-06"
  page_size: 10
```

## Critical Notes

**1. User OAuth is the recommended execution path**

A Server-to-Server OAuth token can:
- initialize against the MCP gateway
- complete `tools/list`
- open SSE sessions

Use user OAuth as the default execution path for Zoom MCP tool use unless you have already
validated S2S scope coverage and tool execution for your app.

**2. Zoom MCP uses MCP-specific granular scopes**

The Zoom MCP scope set is not the same as the older broad REST scopes.
The key scopes for this surface are:
- `meeting:read:search`
- `meeting:read:assets`
- `cloud_recording:read:list_user_recordings`
- `cloud_recording:read:content`
- `docs:write:import` for Zoom Docs creation

**3. AI Companion features are feature prerequisites, not scope substitutes**

Semantic meeting search, meeting assets, and recording-content retrieval depend on account
features such as **Smart Recording** and **Meeting Summary** for useful results. These feature
settings do not replace the required OAuth scopes.

**4. Whiteboard is a separate MCP surface**

The Zoom MCP endpoint and the Whiteboard MCP endpoint are separate. Route Whiteboard-specific
requests to [whiteboard/SKILL.md](whiteboard/SKILL.md).

**5. Use REST for deterministic meeting CRUD**

The current Zoom MCP tool surface does not expose deterministic
meeting create, update, or delete tools. If the user needs explicit meeting CRUD operations,
route to [../rest-api/SKILL.md](../rest-api/SKILL.md).

## Server Endpoints

| Transport | URL |
|-----------|-----|
| Streamable HTTP (recommended) | `https://mcp-us.zoom.us/mcp/zoom/streamable` |
| SSE (fallback) | `https://mcp-us.zoom.us/mcp/zoom/sse` |

Whiteboard child skill:
- [whiteboard/SKILL.md](whiteboard/SKILL.md)

## Search and Retrieval Model

`search_meetings` uses AI Companion retrieval rather than a plain metadata filter. In this
use the live MCP server as authoritative for response schema and scope behavior.

Two result families matter most:

- **Recap-oriented results**: AI summary, meeting-linked documents, recordings, and related assets
- **Recording-oriented results**: cloud recording references and transcript-capable resources

Use [examples/transcript-retrieval.md](examples/transcript-retrieval.md) for the main retrieval
workflow.

## Tool Catalog

| Tool | Key Parameters | Required Scope |
|------|---------------|----------------|
| `create_new_file_with_markdown` | `content`*, `file_name`, `parent_id` | `docs:write:import` |
| `get_meeting_assets` | `meetingId`* | `meeting:read:assets` |
| `search_meetings` | `q`, `from`, `to`, `page_size`, `next_page_token` | `meeting:read:search` |
| `get_recording_resource` | `meetingId`*, `types`, `clip_num`, `play_time`, `raw_passcode`, `encode_passcode` | `cloud_recording:read:content` |
| `recordings_list` | `userId`*, `from`, `to`, `meeting_id`, `trash`, `trash_type`, `page_size`, `next_page_token` | `cloud_recording:read:list_user_recordings` |

\* Required parameter

Full parameter and output guidance: [references/tools.md](references/tools.md)

## Key Workflows

**Search meeting content, then retrieve assets:**
```text
search_meetings
  q: "Q4 planning discussion"
  from: "2026-03-01"
  to: "2026-03-06"
→ choose a returned meeting
→ get_meeting_assets  meetingId: "MEETING_ID_OR_UUID"
```

**List recordings, then retrieve recording resources:**
```text
recordings_list
  userId: "me"
  from: "2026-03-01"
  to: "2026-03-06"
→ choose a recording target
→ get_recording_resource  meetingId: "MEETING_UUID_OR_RECORDING_ID"
```

**Create a Zoom Doc from Markdown:**
```text
create_new_file_with_markdown
  file_name: "Q4 Planning Notes"
  content: "# Decisions\n\n- ..."
```

## Error Reference

| Code | Meaning | Fix |
|------|---------|-----|
| `401 Unauthorized` | Missing or rejected bearer token at the endpoint | Re-register the MCP server with a valid bearer token |
| `-32001 Invalid access token` | Token expired, malformed, or missing required scopes | Refresh OAuth token and verify the MCP-specific scopes |
| `-32602 Can not found tool` | Requested tool name is not exposed by the active MCP server | Re-run `tools/list` and use the current tool names for that endpoint |
| `404` | Possible downstream resource-not-found response | Re-discover the target with `search_meetings` or `recordings_list` |

Full error reference: [references/error-codes.md](references/error-codes.md)

## Documentation

### Concepts
- [concepts/mcp-architecture.md](concepts/mcp-architecture.md) — MCP protocol, hosted endpoints, discovery, and capability model
- [concepts/oauth-setup.md](concepts/oauth-setup.md) — OAuth app creation, MCP-specific scopes, AI Companion prerequisites, token lifecycle

### Examples
- [examples/transcript-retrieval.md](examples/transcript-retrieval.md) — Search/assets and recording-resource workflows
- [examples/create-zoom-doc.md](examples/create-zoom-doc.md) — Verified Zoom Docs creation flow
- [examples/search-and-act.md](examples/search-and-act.md) — Search, inspect assets, and hand off CRUD work to REST when needed
- [examples/meeting-lifecycle.md](examples/meeting-lifecycle.md) — Why meeting CRUD belongs in REST, plus the MCP-to-REST handoff pattern

### References
- [references/tools.md](references/tools.md) — Current Zoom MCP tool reference
- [references/error-codes.md](references/error-codes.md) — MCP and Zoom API errors with fixes
- [whiteboard/SKILL.md](whiteboard/SKILL.md) — Whiteboard MCP child skill

### Troubleshooting
- [troubleshooting/common-errors.md](troubleshooting/common-errors.md) — Scope failures, endpoint mixups, search/recording issues

### Operations
- [RUNBOOK.md](RUNBOOK.md) — 5-minute preflight and debugging checklist

## Related Skills

- [zoom-rest-api](../rest-api/SKILL.md) — Deterministic REST API access, including meeting CRUD
- [zoom-oauth](../oauth/SKILL.md) — OAuth implementation patterns
- [zoom-webhooks](../webhooks/SKILL.md) — Event-driven recording and meeting workflows
- [zoom-rtms](../rtms/SKILL.md) — Live media and transcript streams during active meetings
