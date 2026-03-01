# Tool Reference — Zoom MCP Server

All 12 tools available on the Zoom MCP Server. Always use fully qualified names:
`zoom-mcp:tool_name`

Use `zoom-mcp:get_tool_details` with any `toolName` to retrieve the live parameter schema
directly from the server.

---

## Meeting Management

### `zoom-mcp:search_meetings`

Search meetings by topic, participant, or date range.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Search term (topic, participant name) |
| `startDate` | string | No | Start of date range (`YYYY-MM-DD`) |
| `endDate` | string | No | End of date range (`YYYY-MM-DD`, max 30-day window) |
| `pageSize` | integer | No | Results per page (max 300) |
| `nextPageToken` | string | No | Token for next page |

---

### `zoom-mcp:list_meetings`

List meetings for the authenticated user.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | No | `scheduled`, `live`, or `upcoming` (default: `scheduled`) |
| `pageSize` | integer | No | Results per page (max 300) |
| `nextPageToken` | string | No | Token for next page |

---

### `zoom-mcp:get_meeting`

Get details for a specific meeting.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `meetingId` | string | **Yes** | Meeting ID |
| `occurrenceId` | string | No | Occurrence ID for recurring meetings |
| `showPreviousOccurrences` | boolean | No | Include past occurrences |

---

### `zoom-mcp:create_meeting`

Create a new Zoom meeting.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `topic` | string | **Yes** | Meeting topic/title |
| `type` | integer | **Yes** | `1`=instant, `2`=scheduled, `3`=recurring (no fixed time), `8`=recurring (fixed time) |
| `startTime` | string | No | ISO 8601 datetime (e.g., `2025-03-10T14:00:00`) |
| `duration` | integer | No | Duration in minutes |
| `timezone` | string | No | Timezone (e.g., `America/New_York`) |
| `agenda` | string | No | Meeting description |
| `recurrence` | object | No | Required for type 3 or 8 (see below) |
| `settings` | object | No | Meeting settings (see below) |

**recurrence object:**

| Field | Type | Description |
|-------|------|-------------|
| `type` | integer | `1`=daily, `2`=weekly, `3`=monthly |
| `repeat_interval` | integer | Interval between recurrences |
| `weekly_days` | string | Day(s) of week: `1`=Sun through `7`=Sat |
| `end_times` | integer | Number of occurrences |
| `end_date_time` | string | End date in ISO 8601 |

**settings object:**

| Field | Type | Description |
|-------|------|-------------|
| `host_video` | boolean | Start with host video on |
| `participant_video` | boolean | Start with participant video on |
| `join_before_host` | boolean | Allow joining before host |
| `mute_upon_entry` | boolean | Mute participants on entry |
| `waiting_room` | boolean | Enable waiting room |
| `auto_recording` | string | `local`, `cloud`, or `none` |

---

### `zoom-mcp:update_meeting`

Update an existing meeting. Accepts the same parameters as `create_meeting` plus:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `meetingId` | string | **Yes** | ID of the meeting to update |

Only fields you pass are changed — omitted fields keep their current values.

---

### `zoom-mcp:delete_meeting`

Delete a meeting.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `meetingId` | string | **Yes** | Meeting ID to delete |
| `occurrenceId` | string | No | Delete only this recurrence occurrence |
| `scheduleForReminder` | boolean | No | Send cancellation notification |

---

## Recordings & Transcripts

### `zoom-mcp:list_recordings`

List cloud recordings for the authenticated user.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `from` | string | **Yes** | Start date (`YYYY-MM-DD`) |
| `to` | string | **Yes** | End date (`YYYY-MM-DD`, max 30-day window) |
| `pageSize` | integer | No | Results per page |
| `nextPageToken` | string | No | Token for next page |
| `trash` | boolean | No | List trashed recordings |

---

### `zoom-mcp:get_recording`

Get recording details for a specific meeting, including all recording files.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `meetingId` | string | **Yes** | Meeting ID (or recording UUID) |

**Recording file types in `recording_files`:**

| `file_type` | Description |
|-------------|-------------|
| `MP4` | Video recording |
| `M4A` | Audio-only recording |
| `VTT` | WebVTT transcript with timestamps |
| `TRANSCRIPT` | Structured transcript with speaker labels |
| `CHAT` | In-meeting chat messages |
| `TIMELINE` | Meeting timeline events |

Access transcript content by fetching `download_url` with `Authorization: Bearer TOKEN`.

---

### `zoom-mcp:delete_recording`

Delete or trash a cloud recording.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `meetingId` | string | **Yes** | Meeting ID |
| `action` | string | No | `trash` (default) or `delete` (permanent) |

---

## User & Utility

### `zoom-mcp:get_user_profile`

Get the authenticated user's Zoom profile. No parameters required.

Returns: `id`, `email`, `first_name`, `last_name`, `account_id`, `type`, `timezone`.

---

### `zoom-mcp:list_available_tools`

List all tools available on this MCP server. No parameters required.

Use to confirm server connectivity and discover the current tool list.

---

### `zoom-mcp:get_tool_details`

Get the full parameter schema for any tool.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `toolName` | string | **Yes** | Tool name (e.g., `search_meetings`) |

Returns the complete JSON Schema for the tool's input parameters, directly from the server.
