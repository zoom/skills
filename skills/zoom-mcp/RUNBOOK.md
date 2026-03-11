# zoom-mcp RUNBOOK — 5-Minute Preflight

Quick diagnostic checklist before using the Zoom MCP server.

## Preflight Checklist

**1. MCP server registered?**
```bash
claude mcp list
# Should show: zoom-mcp -> https://mcp-us.zoom.us/mcp/zoom/streamable
```
If missing, re-add it using [concepts/oauth-setup.md](concepts/oauth-setup.md).

**2. Tool discovery working?**
- Confirm the client can see `recordings_list`, `search_meetings`, `get_meeting_assets`,
  `get_recording_resource`, and `create_new_file_with_markdown`.
- If your client exposes raw protocol inspection, verify `tools/list` succeeds.
- Compare the visible tools with [references/tools.md](references/tools.md).

**3. Correct OAuth scopes on the token?**

Minimum Zoom MCP scopes for this guide:
- `meeting:read:search`
- `meeting:read:assets`
- `cloud_recording:read:list_user_recordings`
- `cloud_recording:read:content`
- `docs:write:import` if you want Zoom Docs creation

Whiteboard uses a separate scope set. See [whiteboard/SKILL.md](whiteboard/SKILL.md).

**4. AI Companion features enabled?**

Go to Zoom web portal → **Admin → Account Management → Account Settings → AI Companion**.
Confirm **Smart Recording** and **Meeting Summary** are enabled if you expect semantic search,
meeting assets, or transcript-rich recording content to be useful.

## Quick Fixes

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| `-32001 Access token is required` | Header not passed | Re-register the MCP server with a bearer token |
| `-32001 Invalid access token, does not contain scopes:[meeting:read:search]` | Missing semantic-search scope | Add `meeting:read:search` and mint a new user token |
| `-32001 Invalid access token, does not contain scopes:[meeting:read:assets,...]` | Missing meeting-assets scope | Add `meeting:read:assets` and mint a new user token |
| `-32001 Invalid access token, does not contain scopes:[cloud_recording:read:list_user_recordings,...]` | Missing recordings-list scope | Add `cloud_recording:read:list_user_recordings` |
| `-32001 Invalid access token, does not contain scopes:[cloud_recording:read:content]` | Missing recording-content scope | Add `cloud_recording:read:content` |
| `-32602 Can not found tool: ... in this MCP Server` | Wrong endpoint surface or wrong tool name | Re-run `tools/list` and use the current tool names for the registered MCP server |
| `-32603 Call handle error` | Missing required parameters or server-side call handling failure | Re-check required arguments against the live schema and retry |
| `Upstream API returned error status code: 400 ... invalid param` | Invalid parameter value passed through to the underlying Zoom API | Fix the specific argument value, such as `parent_id` for Docs creation |
| Search returns no useful meeting content | AI Companion features missing or data not indexed | Enable Smart Recording + Meeting Summary, widen the search window, or fall back to `recordings_list` |

## S2S Reality Check

S2S can establish transport and discovery for:
- `initialize`
- `tools/list`
- SSE session establishment

Use user OAuth as the default execution path for Zoom MCP content tools unless you have
already validated S2S scope coverage and tool execution for your app.

## Where to Get Help

- Developer forum: https://devforum.zoom.us/
- Zoom support: https://support.zoom.com/
- MCP protocol docs: https://modelcontextprotocol.io/
