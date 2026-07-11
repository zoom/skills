# Meetings MCP Tools

| Need | Tool | Notes |
|------|------|-------|
| Search by topic, participant, or date | `search_meetings` | Semantic search; results depend on licensed AI/meeting assets. |
| Enumerate assets for a known meeting | `get_meeting_assets` | Can return summaries, recordings, Whiteboards, Docs, and agenda assets. |
| List the authorized user's recordings | `recordings_list` | Requires Pro or higher and cloud recording enabled. |
| Retrieve transcript/summary/playback resources | `get_recording_resource` | Use after confirming the meeting has recording content. |

Re-run `tools/list` before relying on a cached schema. Use the REST API skill when exact request
control, bulk pagination, retry policy, or meeting create/update/delete operations are required.
