---
name: translator
description: "Zoom AI Services Translator for synchronous text translation and asynchronous batch file translation. Use for plain-text translation, one-target-language jobs, S3 text archives, Build-platform JWT auth, and webhook-driven translation pipelines."
triggers:
  - "translator"
  - "ai services translator"
  - "zoom translator"
  - "translate text"
  - "batch translation"
  - "fast mode translation"
  - "text translation api"
  - "build platform jwt"
---

# Zoom AI Services Translator

Implementation guidance for Zoom AI Services Translator across:
- synchronous plain-text translation (`POST /aiservices/translator/translate`)
- asynchronous batch text-file jobs (`/aiservices/translator/jobs*`)
- one-target-language translation workflows
- Build-platform JWT generation and server-side credential handling
- webhook-driven batch status updates

Official docs:
- https://developers.zoom.us/docs/ai-services/
- https://developers.zoom.us/docs/ai-services/translator/
- https://developers.zoom.us/docs/ai-services/translator/fast-mode/
- https://developers.zoom.us/docs/ai-services/translator/batch-mode/
- https://developers.zoom.us/api-hub/ai-services/methods/endpoints.json
- Quickstart sample: https://github.com/zoom/ai-services-quickstart/

## Routing Guardrail

- If the user needs **plain text translated**, route here first.
- If the user needs **audio/video transcribed before translation**, route to [../scribe/SKILL.md](../scribe/SKILL.md), then chain here.
- If the user needs **summaries or action items**, route to [../summarizer/SKILL.md](../summarizer/SKILL.md).
- If the user needs **live meeting media** before translation, route to [../rtms/SKILL.md](../rtms/SKILL.md).
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
3. Choose **fast mode** for one text string or **batch mode** for stored text files.
4. Set `source_language` and a single `target_languages` entry.
5. Submit the request and persist `result.translations`.
6. For batch jobs, poll job/file status or receive webhook notifications.

## Endpoint Surface

| Mode | Method | Path | Use |
|------|--------|------|-----|
| Fast | `POST` | `/aiservices/translator/translate` | Synchronous translation for one text input |
| Batch | `POST` | `/aiservices/translator/jobs` | Submit asynchronous batch translation job |
| Batch | `GET` | `/aiservices/translator/jobs` | List batch translation jobs |
| Batch | `GET` | `/aiservices/translator/jobs/{jobId}` | Inspect job summary/state |
| Batch | `DELETE` | `/aiservices/translator/jobs/{jobId}` | Cancel queued/processing job |
| Batch | `GET` | `/aiservices/translator/jobs/{jobId}/files` | Inspect per-file results |
| Batch | `GET` | `/aiservices/translator/jobs/{jobId}/files/{fileId}` | Inspect one per-file result |

## Mode Selection

| Need | Use |
|------|-----|
| One text value under `4,000` characters | Fast mode |
| `.txt` files in S3 | Batch mode |
| Chat messages, UI strings, workflow actions | Fast mode |
| Large offline document archive | Batch mode |

## Translation Guardrails

- Fast mode supports one target language per request.
- Batch mode supports one target language per job.
- Translation is bidirectional with English: every job must have either `source_language` or `target_languages` set to `en-US`.
- Non-English to non-English pairs are not supported yet.
- Use BCP-47 locale codes such as `en-US` and `es-ES`, not short codes such as `en`.

## Chaining

- Media to transcript to translation -> [../scribe/SKILL.md](../scribe/SKILL.md) + `translator`
- Summary to localized output -> [../summarizer/SKILL.md](../summarizer/SKILL.md) + `translator`
- Batch webhook receiver -> `translator` + [../webhooks/SKILL.md](../webhooks/SKILL.md)
- Raw endpoint discovery -> [../rest-api/SKILL.md](../rest-api/SKILL.md)
- Cross-product routing -> [../general/SKILL.md](../general/SKILL.md)

## Operations

- [RUNBOOK.md](RUNBOOK.md) - 5-minute preflight and debugging checklist.
