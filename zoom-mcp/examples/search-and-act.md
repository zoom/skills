# Search and Act — Finding Meetings and Taking Action

Patterns for searching meeting content and metadata, then acting on results.

`search_meetings` uses AI Companion's Agentic Retrieval API — it searches what was
actually said and discussed, not just meeting titles. Recap results include inline AI
summaries and any Zoom Doc URLs attached to the meeting. See
[examples/transcript-retrieval.md](transcript-retrieval.md) for the full retrieval workflow.

---

## Search by Topic

```
zoom-mcp:search_meetings
  query: "budget review"
  pageSize: 10
```

Returns meetings whose topic matches the query. Results include meeting ID, topic, start
time, host, and status.

---

## Search by Date Range

```
zoom-mcp:search_meetings
  query: "all hands"
  startDate: "2025-01-01"
  endDate: "2025-01-31"
  pageSize: 300
```

Date format: `YYYY-MM-DD`. Maximum date window: 30 days per query. For longer ranges,
run multiple queries with 30-day windows.

Maximum `pageSize`: 300. Use `nextPageToken` from the response to paginate.

---

## Search Then Get Details

A two-step pattern to find and inspect a meeting:

```
Step 1:
zoom-mcp:search_meetings
  query: "product roadmap"
  startDate: "2025-02-01"
  endDate: "2025-02-28"

Step 2 (use ID from results):
zoom-mcp:get_meeting
  meetingId: "84574185634"
```

`get_meeting` returns the full meeting object including join URL, password, recurrence
details, and settings — more than `search_meetings` returns alone.

---

## Search Then Reschedule

```
Step 1: zoom-mcp:search_meetings  →  find the meeting ID

Step 2:
zoom-mcp:update_meeting
  meetingId: "84574185634"
  startTime: "2025-03-20T14:00:00"
  timezone: "America/Chicago"
```

---

## Search Then Get Recording

Find a past meeting and retrieve its recording/transcript:

```
Step 1: zoom-mcp:search_meetings  →  get meeting ID

Step 2:
zoom-mcp:get_recording
  meetingId: "84574185634"
```

If the recording isn't found by meeting ID (common for past meetings where the UUID changes
post-completion), use `list_recordings` with a date range instead:

```
zoom-mcp:list_recordings
  from: "2025-02-01"
  to: "2025-02-28"
  pageSize: 50
```

---

## Search Then Delete

Find and cancel a meeting:

```
Step 1: zoom-mcp:search_meetings  →  find the meeting

Step 2:
zoom-mcp:delete_meeting
  meetingId: "84574185634"
```

---

## Get User Context

Before searching, optionally confirm whose account you're operating on:

```
zoom-mcp:get_user_profile
```

Returns name, email, account type, and timezone — useful for setting defaults in subsequent
create/update calls.

---

## Example Prompts

**Find and summarize:**
> "Find all meetings about 'product launch' in February and tell me when they were."

**Find and reschedule:**
> "Find my client onboarding meeting and move it to next Monday at 10am."

**Find recording:**
> "Get the recording from last week's all-hands meeting."

**Audit:**
> "List all meetings I scheduled in January 2025."
