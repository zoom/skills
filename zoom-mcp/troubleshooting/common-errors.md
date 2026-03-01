# Common Errors — Zoom MCP Server

Diagnostic guide for the most frequent issues when using the Zoom MCP Server with Claude.

---

## Authentication Issues

### "Access token is required" (`-32001`)

**Symptom**: Every tool call returns `-32001 Access token is required`.

**Cause**: The Authorization header was not included when registering the MCP server, or
the registration is missing.

**Fix**:
```bash
# Remove existing registration (if any)
claude mcp remove zoom-mcp

# Re-add with Bearer token
claude mcp add --transport http zoom-mcp \
  https://mcp-us.zoom.us/mcp/zoom/streamable \
  --header "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  --scope user
```

---

### "Invalid access token" (`-32001`)

**Symptom**: The server returns `-32001 Invalid access token`.

**Cause**: The OAuth access token has expired (tokens expire after 1 hour) or was revoked.

**Fix**:
1. Refresh the token:
   ```bash
   curl -X POST https://zoom.us/oauth/token \
     -u "CLIENT_ID:CLIENT_SECRET" \
     -d "grant_type=refresh_token&refresh_token=YOUR_REFRESH_TOKEN"
   ```
2. Re-register with the new access token:
   ```bash
   claude mcp remove zoom-mcp
   claude mcp add --transport http zoom-mcp \
     https://mcp-us.zoom.us/mcp/zoom/streamable \
     --header "Authorization: Bearer NEW_ACCESS_TOKEN" \
     --scope user
   ```

---

## Transcript Issues

### Transcript not in `recording_files`

**Symptom**: `get_recording` returns a `recording_files` array but it contains only MP4/M4A
entries — no VTT or TRANSCRIPT file.

**Most common cause**: AI Companion is not enabled on the account.

**Fix**:
1. Go to Zoom web portal → **Admin → Account Management → Account Settings → AI Companion**
2. Enable **Smart Recording** and **Meeting Summary**
3. For future meetings: these settings apply automatically
4. For past meetings: transcripts cannot be retroactively generated; only new recordings
   after enabling will include transcript files

**Secondary cause**: The meeting was recorded locally, not to cloud.
- Only cloud recordings generate transcript files
- Update the meeting: `zoom-mcp:update_meeting` with `settings: { auto_recording: "cloud" }`

---

### Recording file download returns 401

**Symptom**: The `download_url` from `get_recording` returns `401 Unauthorized` when fetched.

**Cause**: The download URL requires OAuth authentication but no token was passed.

**Fix**: Include the Authorization header when fetching the file:
```
Authorization: Bearer YOUR_ZOOM_OAUTH_TOKEN
```

The same token used for MCP calls is required for file downloads.

---

## Connection Issues

### "instance not found" (`-32602`)

**Symptom**: Error on any tool call: `-32602 instance not found`.

**Cause**: The registered URL points to the Whiteboard MCP server instead of the Zoom
meeting/recording server.

**Fix**: Confirm the URL:
- ✅ Correct: `https://mcp-us.zoom.us/mcp/zoom/streamable`
- ❌ Wrong: `https://mcp-us.zoom.us/mcp/whiteboard/streamable`

---

### MCP server not appearing in Claude

**Symptom**: Claude doesn't offer zoom-mcp tools or says the server isn't registered.

**Fix**:
```bash
# Check registration
claude mcp list

# If not listed, add it
claude mcp add --transport http zoom-mcp \
  https://mcp-us.zoom.us/mcp/zoom/streamable \
  --header "Authorization: Bearer YOUR_ZOOM_OAUTH_TOKEN" \
  --scope user
```

---

## Meeting / Recording Not Found

### `404` on `get_recording`

**Symptom**: `get_recording` returns 404 even though you know the recording exists.

**Cause**: Meeting IDs for past meetings may differ from the original scheduled meeting ID
after the meeting completes. Zoom sometimes uses a UUID-based ID for the recording.

**Fix**: Use `list_recordings` with a date range instead:
```
zoom-mcp:list_recordings
  from: "2025-02-01"
  to: "2025-02-28"
```

The response includes the correct `uuid` to pass to `get_recording`.

---

### Empty search results

**Symptom**: `search_meetings` returns no results for a topic you know exists.

**Causes and fixes**:
1. **Date range too narrow**: Widen `startDate`/`endDate`
2. **Topic spelling**: Try a shorter search term or partial match
3. **Meeting belongs to another host**: `search_meetings` only returns meetings the
   authenticated user hosts or is invited to
4. **Past meetings not indexed**: Some older past meetings may not appear in search;
   use `list_recordings` by date range for past content

---

## Rate Limits

### `429 Too Many Requests`

**Symptom**: Occasional failures with status `429`.

**Fix**: Back off and retry after the interval specified in the `Retry-After` response header.
The default rate limit is approximately 10 requests/second. Avoid rapid sequences of large
paginated queries.
