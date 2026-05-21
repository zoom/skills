---
name: summarizer
description: "Zoom AI Services Summarizer for synchronous transcript summarization and asynchronous batch summary jobs. Use for conversation recaps, action items, detailed summaries, Build-platform JWT auth, S3 transcript archives, and webhook-driven summary pipelines."
triggers:
  - "summarizer"
  - "ai services summarizer"
  - "zoom summarizer"
  - "summarize transcript"
  - "meeting recap api"
  - "extract action items"
  - "batch summarization"
  - "fast mode summarization"
  - "build platform jwt"
---

# Zoom AI Services Summarizer

Implementation guidance for Zoom AI Services Summarizer across:
- synchronous inline transcript summarization (`POST /aiservices/summarizer/summarize`)
- asynchronous batch transcript-file jobs (`/aiservices/summarizer/jobs*`)
- recap, action item, detailed summary, and combined summary tasks
- Build-platform JWT generation and server-side credential handling
- webhook-driven batch status updates

Official docs:
- https://developers.zoom.us/docs/ai-services/
- https://developers.zoom.us/docs/ai-services/summarizer/
- https://developers.zoom.us/docs/ai-services/summarizer/fast-mode/
- https://developers.zoom.us/docs/ai-services/summarizer/batch-mode/
- https://developers.zoom.us/api-hub/ai-services/methods/endpoints.json
- Quickstart sample: https://github.com/zoom/ai-services-quickstart/

## Routing Guardrail

- If the user already has **transcript or dialogue text** and needs a recap, summary, action items, or full summary, route here first.
- If the user has **audio/video media** and needs text first, route to [../scribe/SKILL.md](../scribe/SKILL.md), then chain here.
- If the user needs **plain text translated**, route to [../translator/SKILL.md](../translator/SKILL.md).
- If the user needs **live meeting media** before summarization, route to [../rtms/SKILL.md](../rtms/SKILL.md).
- If the user needs the raw AI Services endpoint inventory, chain [../rest-api/SKILL.md](../rest-api/SKILL.md).
- If the user needs webhook signature patterns, optionally chain [../webhooks/SKILL.md](../webhooks/SKILL.md).

## Quick Links

1. [scenarios/high-level-scenarios.md](scenarios/high-level-scenarios.md)
2. [examples/fast-mode-node.md](examples/fast-mode-node.md)
3. [examples/batch-webhook-pipeline.md](examples/batch-webhook-pipeline.md)
4. [references/api-reference.md](references/api-reference.md)
5. [references/environment-variables.md](references/environment-variables.md)
6. [references/samples-validation.md](references/samples-validation.md)
7. [troubleshooting/common-drift-and-breaks.md](troubleshooting/common-drift-and-breaks.md)
8. [RUNBOOK.md](RUNBOOK.md)

## Core Workflow

1. Get Build-platform API key and API secret.
2. Generate an HS256 JWT server-side.
3. Choose **fast mode** for inline transcript text or **batch mode** for stored transcript files.
4. Select the summarization `task`: `recap`, `summary`, `action_items`, or `full_summary`.
5. Submit the request and persist `result.text` or batch output files.
6. For batch jobs, poll job/file status or receive webhook notifications.

## Endpoint Surface

| Mode | Method | Path | Use |
|------|--------|------|-----|
| Fast | `POST` | `/aiservices/summarizer/summarize` | Synchronous summarization for inline transcript text |
| Batch | `POST` | `/aiservices/summarizer/jobs` | Submit asynchronous batch summarization job |
| Batch | `GET` | `/aiservices/summarizer/jobs` | List batch summarization jobs |
| Batch | `GET` | `/aiservices/summarizer/jobs/{jobId}` | Inspect job summary/state |
| Batch | `DELETE` | `/aiservices/summarizer/jobs/{jobId}` | Cancel queued/processing job |
| Batch | `GET` | `/aiservices/summarizer/jobs/{jobId}/files` | Inspect per-file results |
| Batch | `GET` | `/aiservices/summarizer/jobs/{jobId}/files/{fileId}` | Inspect one per-file result |

## Mode Selection

| Need | Use |
|------|-----|
| One bounded transcript string under `96 KB` | Fast mode |
| Many `.vtt`, `.srt`, or `.txt` transcript files in S3 | Batch mode |
| Summary immediately in app UI | Fast mode |
| Archive or compliance pipeline | Batch mode plus polling/webhooks |
| Audio/video input | `scribe` first, then `summarizer` |

## Output Contract

Summarizer returns rendered text in `result.text`. Treat it as display-ready text or markdown-like content, depending on the requested task:
- `recap`: concise recap
- `summary`: detailed summary
- `action_items`: action item sections grouped by owner
- `full_summary`: recap, summary, and action items together

## Chaining

- Media to transcript to summary -> [../scribe/SKILL.md](../scribe/SKILL.md) + `summarizer`
- Summary to localized output -> `summarizer` + [../translator/SKILL.md](../translator/SKILL.md)
- Batch webhook receiver -> `summarizer` + [../webhooks/SKILL.md](../webhooks/SKILL.md)
- Raw endpoint discovery -> [../rest-api/SKILL.md](../rest-api/SKILL.md)
- Cross-product routing -> [../general/SKILL.md](../general/SKILL.md)

## Operations

- [RUNBOOK.md](RUNBOOK.md) - 5-minute preflight and debugging checklist.
