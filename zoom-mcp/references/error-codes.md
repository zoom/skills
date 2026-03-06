# Error Codes — Zoom MCP Server

Errors from the Zoom MCP Server appear at two layers: the MCP protocol layer (JSON-RPC
error codes) and the Zoom API layer (HTTP status codes surfaced in error messages).

---

## MCP Protocol Errors

These appear as `{"error": {"code": N, "message": "..."}}` in the raw JSON-RPC response.

| Code | Message | Cause | Fix |
|------|---------|-------|-----|
| `-32001` | `Access token is required` | No Authorization header sent | Re-run `claude mcp add` with `--header "Authorization: Bearer TOKEN"` |
| `-32001` | `Invalid access token` | Token expired or malformed | Refresh the OAuth token; re-run `claude mcp add` with updated token |
| `-32602` | `instance not found` | Wrong MCP endpoint URL | Confirm URL is `.../mcp/zoom/streamable`, not `.../mcp/whiteboard/...` |
| `-32602` | `Invalid params` | Missing required parameter or wrong format | Check parameter types and required fields |
| `-32603` | `Internal error` | Server-side error | Retry; if persistent, check [devforum.zoom.us](https://devforum.zoom.us/) |

---

## Zoom API Errors (surfaced through MCP)

These appear in the error message after the MCP server processes your request.

| HTTP Status | Common Cause | Fix |
|-------------|-------------|-----|
| `400 Bad Request` | Invalid parameter value or format | Check date formats (`YYYY-MM-DD` or ISO 8601), meeting type integers, required fields |
| `401 Unauthorized` | OAuth token invalid for this operation | Re-authorize; verify scopes on your OAuth app |
| `403 Forbidden` | Insufficient OAuth scope | Add missing scope (e.g., `recording:write`) to your app and re-authorize |
| `404 Not Found` | Meeting or recording ID doesn't exist | Confirm the ID; use `search_meetings` or `list_recordings` to find it |
| `409 Conflict` | Duplicate meeting creation | Check if the meeting was already created |
| `429 Too Many Requests` | Rate limit exceeded | Back off and retry; default limit is ~10 req/sec; check `Retry-After` header |

---

## Recording-Specific Errors

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `recording_files` is empty or missing transcript | AI Companion not enabled | Enable Smart Recording in Zoom admin → Account Settings → AI Companion |
| `recording_files` has no `VTT`/`TRANSCRIPT` entries | Cloud recording not configured | Set `auto_recording: "cloud"` on the meeting; local recordings don't generate transcripts |
| Download URL returns `401` | Token not passed when fetching file | Add `Authorization: Bearer TOKEN` header to the file download request |
| Recording `status` is `processing` | Transcript not yet ready | Wait a few minutes; recordings typically process within 5–10 minutes of meeting end |

---

## Whiteboard Server Confusion

Attempting to use the Whiteboard MCP server (`/mcp/whiteboard/streamable`) without a
Whiteboard instance ID returns:

```json
{"error": {"code": -32602, "message": "instance not found"}}
```

The Zoom MCP skill (`mcp-us.zoom.us/mcp/zoom/streamable`) and the Whiteboard MCP server
(`mcp-us.zoom.us/mcp/whiteboard/streamable`) are separate. This skill covers the Zoom server
only.

---

## Debugging Tips

1. **Verify server is reachable**: Use `zoom-mcp:list_available_tools` — if it returns the
   tool list, the connection and auth are working.

2. **Confirm user identity**: Use `zoom-mcp:get_user_profile` to verify the token is
   authenticating as the correct user.

3. **Check token freshness**: Zoom OAuth access tokens expire after 1 hour. If operations
   were working earlier and now return `-32001`, the token has expired.

4. **Inspect raw error details**: Claude will surface the full error message from the server.
   The JSON-RPC error `message` field often contains the underlying Zoom API error detail.
