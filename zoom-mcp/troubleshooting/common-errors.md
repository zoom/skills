# Common Errors — Zoom MCP Server

Diagnostic guide for the most frequent issues when using the Zoom MCP server.

## Authentication Issues

### `Access token is required` (`-32001`)

**Cause:** The Authorization header was not included when the MCP server was registered, or the
registration is missing.

**Fix:** Re-register the MCP server with a bearer token.

```bash
claude mcp remove zoom-mcp
claude mcp add --transport http zoom-mcp \
  https://mcp-us.zoom.us/mcp/zoom/streamable \
  --header "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  --scope user
```

### `Invalid access token` (`-32001`)

**Cause:** The access token expired, was revoked, or does not contain the exact MCP scopes needed
for the tool you called.

**Fix:** Refresh the token and verify the MCP-specific granular scopes.

## Scope Issues

### `search_meetings` fails on scope

**Server-required scope:** `meeting:read:search`

### `get_meeting_assets` fails on scope

**Server-required scope:** `meeting:read:assets`

### `recordings_list` fails on scope

**Server-required scope:** `cloud_recording:read:list_user_recordings`

### `get_recording_resource` fails on scope

**Server-required scope:** `cloud_recording:read:content`

### `create_new_file_with_markdown` fails on scope

**Required scope:** `docs:write:import`

## Search and Retrieval Issues

### Semantic search returns no useful results

Common causes:
- AI Companion features were not enabled for those meetings
- the query is too narrow
- the date window is too narrow
- the meeting is easier to locate through `recordings_list`

**Fixes:**
- enable Smart Recording and Meeting Summary
- widen `from` and `to`
- try shorter search terms
- fall back to `recordings_list`

### Meeting assets retrieval fails even with a valid token

Most often this means one of these:
- missing `meeting:read:assets`
- guessed `meetingId` instead of the right UUID or numeric ID
- user does not have access to that meeting's assets

### Recording-oriented retrieval fails

If `get_recording_resource` cannot find the target:
- use `recordings_list` first
- use the identifier returned by the live listing flow
- avoid assuming a scheduled meeting number is the same identifier used by the recording flow

### Direct resource download returns `401`

Returned resource URLs still require the bearer token.

**Fix:** Include `Authorization: Bearer YOUR_TOKEN` when fetching the URL directly.

## Connection Issues

### `Can not found tool: ... in this MCP Server` (`-32602`)

**Cause:** The requested tool does not exist on the registered MCP server, or the request was
sent to the wrong MCP surface.

**Fix:**
- Zoom MCP: `https://mcp-us.zoom.us/mcp/zoom/streamable`
- Whiteboard MCP: `https://mcp-us.zoom.us/mcp/whiteboard/streamable`
- re-run `tools/list`
- use the current tool names exposed by that server
- if the request is Whiteboard-specific, route to [../whiteboard/SKILL.md](../whiteboard/SKILL.md)

### MCP server not appearing in the client

**Fix:**
- run `claude mcp list`
- if missing, re-add the server
- confirm the client is using the expected bearer token

## Parameter and Call-Handling Issues

### `Call handle error` (`-32603`)

**Cause:** The tool call was accepted at the protocol layer, but the server failed while
handling the call. In this repo, this was reproduced by calling `recordings_list` without
required arguments.

**Fix:**
- re-check required parameters against the live schema from `tools/list`
- retry with the missing arguments added

### Upstream `400 invalid param`

**Cause:** The MCP tool reached the downstream Zoom API, but one of the passed arguments was
invalid. In this repo, this was reproduced by calling `create_new_file_with_markdown` with a
bogus `parent_id`.

**Fix:**
- remove the bad argument or replace it with a valid value
- retry the tool call
