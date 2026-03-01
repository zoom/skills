# Transcript Retrieval — Primary Use Case

Retrieving meeting transcripts is the core capability of the Zoom MCP Server — it's the
primary workflow behind the Zoom for ChatGPT app. This document walks through the complete
end-to-end pattern.

---

## Prerequisites

Before transcripts are available:
- AI Companion → **Smart Recording** must be enabled (generates VTT/TRANSCRIPT files)
- The meeting must have been recorded to **cloud** (not local)
- The recording must have completed processing (usually within minutes of meeting end)

See [concepts/oauth-setup.md](../concepts/oauth-setup.md#step-4-enable-ai-companion-required-for-transcripts)
for setup instructions.

---

## The Retrieval Workflow

### Step 1: Find the Meeting

If you know the meeting ID, skip to Step 2. Otherwise, search:

```
zoom-mcp:search_meetings
  query: "Q4 planning"
  startDate: "2025-01-01"
  endDate: "2025-01-31"
```

Or list recent meetings:
```
zoom-mcp:list_meetings
  type: scheduled
  pageSize: 20
```

Note the `id` of the target meeting.

---

### Step 2: Get the Recording

```
zoom-mcp:get_recording
  meetingId: "84574185634"
```

The response includes a `recording_files` array. Each entry has a `file_type` field. Look for:

| `file_type` | Contents |
|-------------|---------|
| `VTT` | WebVTT format — includes timestamps, suitable for navigation |
| `TRANSCRIPT` | Structured JSON with speaker labels and timestamps |
| `MP4` | Video recording |
| `M4A` | Audio-only recording |
| `CHAT` | In-meeting chat transcript |
| `TIMELINE` | Meeting timeline events |

---

### Step 3: Read the Transcript

Each recording file entry includes a `download_url`. Fetch it to retrieve the content:

```
fetch: recording_files[file_type == "TRANSCRIPT"].download_url
  Authorization: Bearer YOUR_ZOOM_OAUTH_TOKEN
```

> **Note:** The download URL requires the same OAuth token used for MCP calls. Pass it as
> an Authorization header when fetching the file directly.

---

### Step 4: Analyze the Content

With the transcript retrieved, Claude can:
- Summarize key discussion points and decisions
- Extract action items with owners
- Answer questions about what was said
- Find specific speaker contributions
- Locate timestamps for topics of interest
- Generate follow-up emails or meeting notes

---

## Example Prompts

Ask Claude these prompts after connecting the zoom-mcp server:

**Find and summarize:**
> "Find my meeting about Q4 planning from last week and summarize the key decisions."

**Extract action items:**
> "Get the transcript from yesterday's team standup and list all action items with owners."

**Speaker search:**
> "In the January 15th product review meeting, what did Sarah say about the roadmap?"

**Generate follow-up:**
> "Based on the transcript from this morning's client call, draft a follow-up email
> summarizing what we agreed to."

---

## Troubleshooting

**No `TRANSCRIPT` or `VTT` in `recording_files`:**
1. Check that AI Companion → Smart Recording is enabled in Zoom account settings
2. Confirm the meeting was recorded to cloud, not local
3. Wait a few minutes — transcript processing takes time after meeting end
4. Check `status` field in `get_recording` response: should be `completed`

**`404` on `get_recording`:**
- The `meetingId` for past meetings may differ from the scheduled ID
- Use `list_recordings` with `from`/`to` dates to find the correct recording UUID

**Download URL returns 401:**
- The OAuth token must be included as `Authorization: Bearer TOKEN` when fetching files
- Tokens expire after 1 hour — refresh if needed
