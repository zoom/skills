# Search Zoom Chat, Docs, and My Notes

Use this Zoom MCP path when the user asks for knowledge discovery across Team Chat messages,
Zoom Docs, or Zoom AI Companion notes/My Notes.

## Prerequisites

- `ai_companion:read:search` for `search_zoom`
- `docs:read:export` for `get_file_content` when reading returned Zoom Docs/My Notes content

## Search Team Chat Messages

```text
search_zoom
  query: "customer escalation"
  search_entities:
    - entity_type: "chat"
      filters:
        channel_names:
          - "support-war-room"
        from: "2026-03-01T00:00:00Z"
        to: "2026-03-06T23:59:59Z"
  page_size: 10
```

Notes:
- Use `entity_type: "chat"` for Team Chat messages.
- Chat filters support `channel_names`, `hsession_ids`, `from`, `to`, and `unread`.
- Convert local time references to ISO 8601 UTC before calling the tool.

## Search Zoom Docs

```text
search_zoom
  query: "launch checklist"
  search_entities:
    - entity_type: "zoom_doc"
      filters:
        doc_view: "my_docs"
        from: "2026-03-01T00:00:00Z"
        to: "2026-03-06T23:59:59Z"
  page_size: 10
```

Supported `doc_view` values:
- `recent`
- `my_docs`
- `shared_with_me`
- `starred`
- `notes`

Use `doc_view: "notes"` when the user specifically asks for My Notes, meeting notes, or
AI-generated notes. Do not force `notes` for generic Zoom Docs searches.

## Read a Returned Doc

After `search_zoom` returns a Zoom Doc result, pass the returned `file_id` to
`get_file_content`.

```text
get_file_content
  fileId: "FILE_ID_FROM_SEARCH_RESULT"
```

The tool returns the file name and Markdown content.

## Choosing the Right Search Tool

| User asks for | Use |
|---------------|-----|
| Meeting by topic, attendee, summary, transcript, or meeting time | `search_meetings` |
| Team Chat messages | `search_zoom` with `entity_type: "chat"` |
| Zoom Docs or My Notes | `search_zoom` with `entity_type: "zoom_doc"` |
| Markdown content of a returned doc/note | `get_file_content` |
