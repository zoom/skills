---
name: zoom-mcp
description: |
  Official Zoom MCP Server guidance for AI-agent access to Zoom search, meeting assets,
  recording resources, Docs, Team Chat, Whiteboard, Tasks, Meetings, and Revenue Accelerator.
  Use for Zoom-hosted MCP endpoints, tools/list, tools/call, OAuth scopes, or selecting the
  right server. Route product-specific work to the matching zoom-mcp child skill.
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
  - "zoom docs content via mcp"
  - "zoom chat search mcp"
  - "team chat mcp"
  - "zoom team chat mcp"
  - "zoom transcript via mcp"
  - "meeting transcript via mcp"
---

# Zoom MCP Server

Zoom hosts an MCP server at `mcp.zoom.us` for AI-agent access to:

- semantic meeting search
- cross-Zoom search over Team Chat messages and Zoom Docs
- meeting-linked asset retrieval
- recording resource retrieval
- Zoom Docs creation from Markdown and Markdown content export
- Zoom Hub file creation and multi-format content export

Current default Zoom MCP server tool names verified by `tools/list`:

- `create_new_file_with_markdown`
- `get_file_content`
- `get_meeting_assets`
- `get_recording_resource`
- `hub_create_file_from_content`
- `hub_get_file_content`
- `recordings_list`
- `search_meetings`
- `search_zoom`

Some MCP clients namespace server tools in the UI, for example `zoom-mcp:recordings_list`.
Treat the raw tool names above as authoritative.

Product-specific MCP work is split into child skills for
[Whiteboard](whiteboard/SKILL.md), [Team Chat](team-chat/SKILL.md),
[Meetings](meetings/SKILL.md), [Docs](docs/SKILL.md), [Tasks](tasks/SKILL.md), and
[Revenue Accelerator](revenue-accelerator/SKILL.md).

> **Marketplace-first skill chain:** Before connecting to any Zoom MCP server, route to
> [Marketplace app management](../rest-api/references/marketplace-apps.md) and the
> [Marketplace template selector](../rest-api/references/marketplace-app-templates.md). Create
> a user-managed General App from the matching MCP template and obtain its client ID. Then
> route to [zoom-oauth](../oauth/SKILL.md) to authorize the user, exchange the authorization
> code (with PKCE where configured), and obtain the bearer access token. Only after those steps
> should this skill configure the MCP endpoint and call `initialize`, `tools/list`, or
> `tools/call`.

## Quick Start

**1. Create the Marketplace app:**

- Select the exact MCP scenario from the
  [Marketplace template selector](../rest-api/references/marketplace-app-templates.md).
- Create the user-managed General App and configure its redirect URI.
- Keep only the scopes required by the MCP tools you intend to call.

**2. Obtain a user OAuth access token:**

- Complete authorization-code OAuth, using PKCE when the client cannot safely hold a secret.
- Exchange the returned code and retain the access and refresh tokens securely.
- Follow [concepts/oauth-setup.md](concepts/oauth-setup.md) for the complete flow.

**3. Add to an MCP client (Claude Code example):**
```bash
claude mcp add --transport http zoom-mcp \
  https://mcp.zoom.us/mcp/zoom/streamable \
  --header "Authorization: Bearer YOUR_ZOOM_OAUTH_TOKEN" \
  --scope user
```

**4. Verify discovery:**
- Confirm the client can see 9 default Zoom MCP tools: `search_meetings`,
  `create_new_file_with_markdown`, `search_zoom`, `get_meeting_assets`,
  `get_recording_resource`, `get_file_content`, `recordings_list`,
  `hub_create_file_from_content`, and `hub_get_file_content`.
- If the client exposes raw protocol inspection, `tools/list` is the authoritative discovery source.
- The current catalog is documented in [references/tools.md](references/tools.md).

**5. Run the first useful call:**
```text
recordings_list
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
- `ai_companion:read:search`
- `cloud_recording:read:list_user_recordings`
- `cloud_recording:read:content`
- `docs:write:import` for Zoom Docs creation
- `docs:read:export` for Zoom Docs/My Notes Markdown content export
- `hub:write:content` for Hub file creation
- `hub:read:content` for Hub file content export

**3. AI Companion features are feature prerequisites, not scope substitutes**

Semantic meeting search, meeting assets, and recording-content retrieval depend on account
features such as **Smart Recording** and **Meeting Summary** for useful results. These feature
settings do not replace the required OAuth scopes.

**4. Whiteboard is a separate MCP surface**

The Zoom MCP endpoint and the Whiteboard MCP endpoint are separate. Route Whiteboard-specific
requests to [whiteboard/SKILL.md](whiteboard/SKILL.md).

**5. Team Chat tools are a separate MCP surface**

The default Zoom MCP server includes read-only `search_zoom` for Team Chat and Docs search.
The Team Chat MCP server is separate and exposes read, search, write, and update tools for
messages, files, contacts, channels, and channel members. Route Team Chat MCP requests to
[team-chat/SKILL.md](team-chat/SKILL.md).

**6. Use REST for deterministic meeting CRUD**

The current Zoom MCP tool surface does not expose deterministic
meeting create, update, or delete tools. If the user needs explicit meeting CRUD operations,
route to [../rest-api/SKILL.md](../rest-api/SKILL.md).

## Server Endpoints

| Transport | URL |
|-----------|-----|
| Streamable HTTP (recommended) | `https://mcp.zoom.us/mcp/zoom/streamable` |
| SSE (fallback) | `https://mcp.zoom.us/mcp/zoom/sse` |

Whiteboard child skill:
- [whiteboard/SKILL.md](whiteboard/SKILL.md)

Team Chat child skill:
- [team-chat/SKILL.md](team-chat/SKILL.md)

Other dedicated child skills:
- [meetings/SKILL.md](meetings/SKILL.md)
- [docs/SKILL.md](docs/SKILL.md)
- [tasks/SKILL.md](tasks/SKILL.md)
- [revenue-accelerator/SKILL.md](revenue-accelerator/SKILL.md)

## Search and Retrieval Model

`search_meetings` uses AI Companion retrieval rather than a plain metadata filter. In this
use the live MCP server as authoritative for response schema and scope behavior.

Two result families matter most:

- **Recap-oriented results**: AI summary, meeting-linked documents, recordings, and related assets
- **Recording-oriented results**: cloud recording references and transcript-capable resources

Use `search_zoom` instead of `search_meetings` when the task is cross-Zoom knowledge discovery
over Team Chat messages, Zoom Docs, or My Notes. Use `get_file_content` after `search_zoom`
when the user asks to inspect the Markdown content of a returned Zoom Doc or My Notes file.

Use [examples/transcript-retrieval.md](examples/transcript-retrieval.md) for the main retrieval
workflow.

## Tool Catalog

| Tool | Key Parameters | Required Scope |
|------|---------------|----------------|
| `create_new_file_with_markdown` | `content`*, `file_name`, `parent_id` | `docs:write:import` |
| `get_file_content` | `fileId`* | `docs:read:export` |
| `get_meeting_assets` | `meetingId`* | `meeting:read:assets` |
| `get_recording_resource` | `meetingId`*, `types`, `clip_num`, `play_time`, `raw_passcode`, `encode_passcode` | `cloud_recording:read:content` |
| `hub_create_file_from_content` | `content`*, file type/name and destination fields from live schema | `hub:write:content` |
| `hub_get_file_content` | file identifier* and `format` | `hub:read:content` |
| `recordings_list` | `from`, `to`, `meeting_id`, `trash`, `trash_type`, `page_size`, `next_page_token` | `cloud_recording:read:list_user_recordings` |
| `search_meetings` | `q`, `from`, `to`, `include_zoom_my_notes`, `page_size`, `next_page_token` | `meeting:read:search` |
| `search_zoom` | `search_entities`*, `query`, `page_size` | `ai_companion:read:search` |

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

**Search Zoom Chat or Docs, then read a returned file:**
```text
search_zoom
  query: "Q4 planning decisions"
  search_entities:
    - entity_type: "zoom_doc"
      filters:
        doc_view: "notes"
  page_size: 10
→ choose a returned Zoom Doc file_id
→ get_file_content  fileId: "FILE_ID"
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
- [examples/search-chat-docs.md](examples/search-chat-docs.md) — Cross-Zoom search over Team Chat, Zoom Docs, and My Notes
- [examples/create-zoom-doc.md](examples/create-zoom-doc.md) — Verified Zoom Docs creation flow
- [examples/search-and-act.md](examples/search-and-act.md) — Search, inspect assets, and hand off CRUD work to REST when needed
- [examples/meeting-lifecycle.md](examples/meeting-lifecycle.md) — Why meeting CRUD belongs in REST, plus the MCP-to-REST handoff pattern

### References
- [references/tools.md](references/tools.md) — Current Zoom MCP tool reference
- [references/error-codes.md](references/error-codes.md) — MCP and Zoom API errors with fixes
- [whiteboard/SKILL.md](whiteboard/SKILL.md) — Whiteboard MCP child skill
- [team-chat/SKILL.md](team-chat/SKILL.md) — Team Chat MCP child skill
- [meetings/SKILL.md](meetings/SKILL.md) — Dedicated Meetings MCP child skill
- [docs/SKILL.md](docs/SKILL.md) — Dedicated Docs MCP child skill
- [tasks/SKILL.md](tasks/SKILL.md) — Tasks MCP child skill
- [revenue-accelerator/SKILL.md](revenue-accelerator/SKILL.md) — Revenue Accelerator MCP child skill

### Troubleshooting
- [troubleshooting/common-errors.md](troubleshooting/common-errors.md) — Scope failures, endpoint mixups, search/recording issues

### Operations
- [RUNBOOK.md](RUNBOOK.md) — 5-minute preflight and debugging checklist

## Related Skills

- [zoom-rest-api](../rest-api/SKILL.md) — Deterministic REST API access, including meeting CRUD
- [zoom-oauth](../oauth/SKILL.md) — OAuth implementation patterns
- [zoom-webhooks](../webhooks/SKILL.md) — Event-driven recording and meeting workflows
- [zoom-rtms](../rtms/SKILL.md) — Live media and transcript streams during active meetings
