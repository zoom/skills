# zoom-mcp RUNBOOK — 5-Minute Preflight

Quick diagnostic checklist before using the Zoom MCP Server with Claude.

---

## Preflight Checklist

**1. Claude MCP server registered?**
```bash
claude mcp list
# Should show: zoom-mcp → https://mcp-us.zoom.us/mcp/zoom/streamable
```
If missing → re-add: see [concepts/oauth-setup.md](concepts/oauth-setup.md)

**2. OAuth token valid?**
```
zoom-mcp:get_user_profile
```
- ✅ Returns your name and email → token is good
- ❌ `-32001 Access token is required` → token missing from header
- ❌ `-32001 Invalid access token` → token expired; generate a new one

**3. AI Companion enabled?** (required for transcripts)

Go to Zoom web portal → **Admin → Account Management → Account Settings → AI Companion**.
Confirm **Smart Recording** and **Meeting Summary** are toggled on.

If not enabled → transcripts and summaries will not appear in `get_recording` results even
when recordings exist.

**4. Correct scopes on your OAuth app?**

Go to [marketplace.zoom.us](https://marketplace.zoom.us) → your app → **Scopes**.
Minimum required:
- `meeting:read` — for search, list, get
- `recording:read` — for list_recordings, get_recording
- `meeting:write` — for create, update, delete

---

## Quick Fixes

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| `-32001 Access token is required` | Header not passed | Re-run `claude mcp add` with `--header "Authorization: Bearer TOKEN"` |
| `-32001 Invalid access token` | Token expired | Re-authorize in Zoom Marketplace; copy new token |
| Transcript file not in `recording_files` | AI Companion disabled | Enable Smart Recording + Meeting Summary in account settings |
| `-32602 instance not found` | Wrong endpoint | Confirm URL is `.../mcp/zoom/streamable` not `.../mcp/whiteboard/...` |
| Empty `recording_files` array | No cloud recording | Check that `auto_recording: "cloud"` was set for the meeting |
| `404` on `get_recording` | Wrong meeting ID | Use `search_meetings` or `list_recordings` to find the correct ID |

---

## Where to Get Help

- **Developer forum**: https://devforum.zoom.us/
- **Zoom support**: https://support.zoom.com/
- **MCP protocol docs**: https://modelcontextprotocol.io/
