# Transcript & Summary Retrieval

Retrieving meeting content is the core capability of the Zoom MCP Server — and the primary
workflow behind the Zoom for ChatGPT app. There are two paths depending on what you need.

---

## Path 1: Agentic Retrieval (Recommended)

The `search_meetings` tool uses AI Companion's **Agentic Retrieval API** to search meeting
content semantically. Results include an inline AI summary — no separate recording fetch
needed for most use cases.

### Prerequisites

- AI Companion → **Smart Recording** and **Meeting Summary** must be enabled
- Meetings must have been recorded to cloud

### Step 1: Search by content or time period

**By content (semantic query):**
```
zoom-mcp:search_meetings
  query: "Q4 budget discussion"
```

**By time period:**
```
zoom-mcp:search_meetings
  startDate: "2025-02-01"
  endDate: "2025-02-28"
  pageSize: 20
```

**Combined:**
```
zoom-mcp:search_meetings
  query: "product roadmap"
  startDate: "2025-01-01"
  endDate: "2025-01-31"
```

### Step 2: Read the Recap meeting result

For meetings with AI Companion data, results include a **Recap meeting** type with:
- **Inline AI summary** — the meeting summary text, returned directly in the response
- **Zoom Doc URLs** — links to any Zoom Docs attached to or created from the meeting
- **Recording reference** — pointer to the cloud recording
- **Whiteboards** — links to any whiteboards from the meeting

> **[CONTRIBUTOR NEEDED]** Exact field names in the Recap meeting result object. Specifically:
> - What is the field name for the inline AI summary text?
> - What is the structure of the docs array and each entry's fields (title, URL)?
> - What is the field name/structure for the recording reference?
> - What is the field name/structure for whiteboards?
> Source: Zoom MCP Server engineering team or live `zoom-mcp:get_tool_details` output.

**View recording** results provide full recording details for meetings with cloud recordings.

> **[CONTRIBUTOR NEEDED]** Exact field names in the View recording result object. What does
> it contain beyond the recording reference in Recap? Source: engineering team or API schema.

### Step 3: Act on the content

With the inline AI summary, Claude can immediately:
- Answer questions about what was discussed
- Extract action items and owners
- Identify decisions made
- Find specific topics or speaker contributions
- Draft follow-up emails or meeting notes
- Create a Zoom Doc with the content → see [examples/create-zoom-doc.md](create-zoom-doc.md)

---

## Path 2: Raw Recording Access (When You Need More)

Use this path when you need:
- Timestamped VTT (exact moment-by-moment captions)
- Speaker-labeled TRANSCRIPT JSON
- The raw video or audio file
- Content from a meeting that predates AI Companion being enabled

### Step 1: Find the recording

**By date range:**
```
zoom-mcp:list_recordings
  from: "2025-02-01"
  to: "2025-02-28"
  pageSize: 50
```

**By meeting ID (from a search result):**
```
zoom-mcp:get_recording
  meetingId: "84574185634"
```

### Step 2: Locate the transcript file

`get_recording` returns a `recording_files` array. Look for:

| `file_type` | Contents |
|-------------|---------|
| `TRANSCRIPT` | Structured JSON with speaker labels and timestamps |
| `VTT` | WebVTT format — timestamps for every caption segment |
| `MP4` | Full video recording |
| `M4A` | Audio-only recording |
| `CHAT` | In-meeting chat messages |

### Step 3: Fetch the transcript

Each file entry has a `download_url`. Fetch it with the OAuth Bearer token:

```
GET {download_url}
Authorization: Bearer YOUR_ZOOM_OAUTH_TOKEN
```

> **Note:** The download URL is not publicly accessible — the Bearer token is required.
> The same token used for MCP calls works for file downloads.

---

## Choosing the Right Path

| If you need... | Use |
|---------------|-----|
| AI-generated summary, key topics, action items | Path 1 — Agentic Retrieval |
| Zoom Doc links from the meeting | Path 1 — Recap result includes URLs |
| Exact timestamps (who said what, when) | Path 2 — VTT file |
| Speaker-attributed full transcript | Path 2 — TRANSCRIPT file |
| Video or audio file | Path 2 — MP4 / M4A |
| Content from before AI Companion was enabled | Path 2 — raw recording |

---

## Example Prompts

**Content search:**
> "Find meetings where we discussed the product launch and summarize the decisions."

**Time period:**
> "What happened in my meetings last week? Give me a summary of the key topics."

**Specific speaker:**
> "In the February all-hands, what did the CEO say about the roadmap?"

**Extract action items:**
> "Get the AI summary from yesterday's team sync and list all action items."

**Full transcript:**
> "I need the word-for-word timestamped transcript from the March 5th client call."

---

## Troubleshooting

**Recap result has no AI summary:**
- AI Companion not enabled → Admin → Account Settings → AI Companion → enable Smart Recording
- Meeting predates AI Companion being turned on → use Path 2 for raw transcript

**No TRANSCRIPT or VTT in `recording_files`:**
- `auto_recording` was not set to `"cloud"` for the meeting
- AI Companion disabled at time of recording

**Download URL returns 401:**
- Include `Authorization: Bearer YOUR_TOKEN` header when fetching the file directly

**`get_recording` returns 404:**
- Use `list_recordings` with a date range — past recording UUIDs differ from meeting IDs
