# Meeting Lifecycle — Create, Update, and Delete

Complete workflows for managing Zoom meetings through the MCP server.

---

## Create a Meeting

### Instant (one-time) meeting

```
zoom-mcp:create_meeting
  topic: "Project kickoff"
  type: 2
  startTime: "2025-03-10T14:00:00"
  duration: 60
  timezone: "America/New_York"
  agenda: "Review scope, assign owners, set next steps"
```

**type values:**
- `1` = instant meeting (starts immediately)
- `2` = scheduled meeting (specific date/time)
- `3` = recurring, no fixed time
- `8` = recurring, fixed time

---

### Recurring weekly meeting

```
zoom-mcp:create_meeting
  topic: "Weekly team sync"
  type: 8
  startTime: "2025-03-03T09:00:00"
  duration: 30
  timezone: "America/New_York"
  recurrence:
    type: 2
    repeat_interval: 1
    weekly_days: "2"
    end_times: 12
  settings:
    auto_recording: "cloud"
    waiting_room: true
    mute_upon_entry: true
```

**recurrence.type values:** `1`=daily, `2`=weekly, `3`=monthly
**weekly_days:** `1`=Sun, `2`=Mon, `3`=Tue, `4`=Wed, `5`=Thu, `6`=Fri, `7`=Sat

---

### Meeting with cloud recording enabled

```
zoom-mcp:create_meeting
  topic: "Client review"
  type: 2
  startTime: "2025-03-15T10:00:00"
  duration: 90
  timezone: "Europe/London"
  settings:
    auto_recording: "cloud"
    host_video: true
    participant_video: true
    join_before_host: false
    waiting_room: true
```

Setting `auto_recording: "cloud"` ensures AI Companion can generate transcripts.

---

## Inspect a Meeting

```
zoom-mcp:get_meeting
  meetingId: "84574185634"
```

Returns full details: join URL, password, settings, recurrence, registrants count.

---

## Update a Meeting

### Reschedule

```
zoom-mcp:update_meeting
  meetingId: "84574185634"
  startTime: "2025-03-17T15:00:00"
  timezone: "America/Los_Angeles"
```

### Change topic and agenda

```
zoom-mcp:update_meeting
  meetingId: "84574185634"
  topic: "Q1 Planning — Updated"
  agenda: "Review updated roadmap and budget"
```

### Enable cloud recording on an existing meeting

```
zoom-mcp:update_meeting
  meetingId: "84574185634"
  settings:
    auto_recording: "cloud"
```

Only fields you pass are updated — omitted fields keep their current values.

---

## List Meetings

```
zoom-mcp:list_meetings
  type: scheduled
  pageSize: 50
```

**type options:** `scheduled`, `live`, `upcoming`

Use `nextPageToken` from the response to paginate.

---

## Delete a Meeting

```
zoom-mcp:delete_meeting
  meetingId: "84574185634"
```

To delete a specific occurrence of a recurring meeting:
```
zoom-mcp:delete_meeting
  meetingId: "84574185634"
  occurrenceId: "1711900800000"
```

---

## Example Prompts

**Schedule:**
> "Create a 45-minute meeting called 'Design Review' for next Tuesday at 2pm Eastern with
> cloud recording enabled."

**Reschedule:**
> "Move my Q1 Planning meeting to Friday at 3pm Pacific."

**Cancel:**
> "Delete the team sync meeting scheduled for March 10th."

**List:**
> "Show me all my scheduled meetings for this week."
