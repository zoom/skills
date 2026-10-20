# Zoom AI Services API

Current OpenAPI inventory for Zoom AI Services, rechecked on 2026-10-20.

## Canonical Source

- OpenAPI JSON: https://developers.zoom.us/api/zoap/ai-services/methods/
- API reference: https://developers.zoom.us/docs/api/ai-services/
- Base URL: `https://api.zoom.us/v2`
- Product skills: [Scribe](../../scribe/SKILL.md), [Summarizer](../../summarizer/SKILL.md), [Speech Summarization](../../speech-summarization/SKILL.md), [Translator](../../translator/SKILL.md), [Decisions](../../decisions/SKILL.md), [Voice Agent](../../voice-agent/SKILL.md)
- Authentication details: [authentication.md](authentication.md)

## Notes

- AI Services accepts an API key through `x-api-key` or `Authorization: Bearer`, and Build-platform key/secret HS256 JWTs remain supported.
- Speech Summarization and Voice Agent are now present in the public OpenAPI catalog.
- The Voice Agent catalog operation is `GET /aiservices/voice-agent/live`, but the official V2 WebSocket sample uses `wss://api.zoom.us/v2/ws/aiservices/voice-agent/live` with `voice-agent.zoom.v2`. Follow the [Voice Agent skill](../../voice-agent/SKILL.md) for the live wire contract.
- A correct request can still fail for project entitlements or credits: `402 QUOTA_EXCEEDED` means no available credits, and a Voice Agent upgrade `404` can mean the project lacks Build Account or Universal Credit provisioning.
- Use this file for endpoint discovery and inventory. Use the product skills for request shapes, streaming semantics, webhooks, and mode selection.

## Coverage

| Metric | Value |
|--------|-------|
| Endpoint operations | 29 |
| Path templates | 21 |
| Tags | 7 |

## Tag Index

| Tag | Operations |
|-----|------------|
| Asset | 4 |
| Decisions | 1 |
| Scribe | 8 |
| Speech Summarization | 1 |
| Summarizer | 7 |
| Translator | 7 |
| VoiceAgent | 1 |

## Endpoints by Tag

### Asset

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/aiservices/assets/vocabularies` | List Vocabularies | `ListVocabularies` |
| POST | `/aiservices/assets/vocabularies` | Create Custom Vocabulary | `CreateCustomVocabulary` |
| GET | `/aiservices/assets/vocabularies/{vocabularyId}` | Get Vocabulary | `GetVocabulary` |
| DELETE | `/aiservices/assets/vocabularies/{vocabularyId}` | Delete Vocabulary | `DeleteVocabulary` |

### Decisions

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/aiservices/decisions/decide` | Evaluate decision questions | `decideDecisions` |

### Scribe

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/aiservices/scribe/jobs` | List Batch Jobs | `listBatchJobs` |
| POST | `/aiservices/scribe/jobs` | Submit Batch Scribe Job | `submitBatchAsr` |
| GET | `/aiservices/scribe/jobs/{jobId}` | Get Batch Job Status | `getBatchJobStatus` |
| DELETE | `/aiservices/scribe/jobs/{jobId}` | Cancel Batch Job | `cancelBatchJob` |
| GET | `/aiservices/scribe/jobs/{jobId}/files` | List Batch Job Files | `listBatchJobFiles` |
| GET | `/aiservices/scribe/jobs/{jobId}/files/{fileId}` | Get Batch Scribe Job File | `getBatchScribeJobFile` |
| GET | `/aiservices/scribe/live` | Live Scribe WebSocket | `liveTranscribe` |
| POST | `/aiservices/scribe/transcribe` | Scribe (Synchronous) | `createFastAsr` |

### Speech Summarization

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/aiservices/speech-summarization/summarize` | Transcribe and summarize one media file over SSE | `transcribeAndSummarizeStream` |

### Summarizer

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/aiservices/summarizer/jobs` | List Batch Summarizer Jobs | `listBatchSummarizerJobs` |
| POST | `/aiservices/summarizer/jobs` | Submit Batch Summarizer Job | `submitBatchSummarizer` |
| GET | `/aiservices/summarizer/jobs/{jobId}` | Get Batch Summarizer Job | `getBatchSummarizerJob` |
| DELETE | `/aiservices/summarizer/jobs/{jobId}` | Cancel Batch Summarizer Job | `cancelBatchSummarizerJob` |
| GET | `/aiservices/summarizer/jobs/{jobId}/files` | List Batch Summarizer Job Files | `listBatchSummarizerJobFiles` |
| GET | `/aiservices/summarizer/jobs/{jobId}/files/{fileId}` | Get Batch Summarizer Job File | `getBatchSummarizerJobFile` |
| POST | `/aiservices/summarizer/summarize` | Summarize (Synchronous) | `summarize` |

### Translator

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/aiservices/translator/jobs` | List Batch Translator Jobs | `listBatchTranslatorJobs` |
| POST | `/aiservices/translator/jobs` | Submit Batch Translator Job | `submitBatchTranslator` |
| GET | `/aiservices/translator/jobs/{jobId}` | Get Batch Translator Job | `getBatchTranslatorJob` |
| DELETE | `/aiservices/translator/jobs/{jobId}` | Cancel Batch Translator Job | `cancelBatchTranslatorJob` |
| GET | `/aiservices/translator/jobs/{jobId}/files` | List Batch Translator Job Files | `listBatchTranslatorJobFiles` |
| GET | `/aiservices/translator/jobs/{jobId}/files/{fileId}` | Get Batch Translator Job File | `getBatchTranslatorJobFile` |
| POST | `/aiservices/translator/translate` | Translate (Synchronous) | `translate` |

### VoiceAgent

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/aiservices/voice-agent/live` | Live Voice Agent WebSocket | `liveVoiceAgent` |

The catalog upgrade operation is listed above, but both working sockets are under:

```text
wss://api.zoom.us/v2/ws/aiservices/voice-agent/live
```

The documented `/v2/aiservices/voice-agent/live` path returns 404. Two subprotocols work on the `/v2/ws/` route:

- `voice-agent.zoom.v2` — official sample; `listen` / `think` / `speak` body; flat events.
- `voice-agent.zoom.v1` — docs live-mode page; OpenAI-style `audio.input` / `audio.output` body; `{type, payload}` events.

Match the body to the subprotocol and authenticate server-side with `x-api-key` or bearer.

## Granular scopes

Scopes below come from each operation's `x-granular-scopes`. Request only what a workflow needs.

| Surface | Operation | Scope |
|---------|-----------|-------|
| Decisions | `POST /aiservices/decisions/decide` | `aiservices:write:decide:admin` |
| Scribe | `POST /aiservices/scribe/transcribe` | `aiservices:write:transcribe:admin` |
| Scribe | `POST /aiservices/scribe/jobs` | `aiservices:write:scribe_batch_job:admin` |
| Scribe | `GET /aiservices/scribe/live` | `aiservices:write:live_transcribe:admin` |
| Speech Summarization | `POST /aiservices/speech-summarization/summarize` | `aiservices:write:transcribe-summarize:admin` |
| Summarizer | `POST /aiservices/summarizer/summarize` | `aiservices:write:summarize:admin` |
| Summarizer | `POST /aiservices/summarizer/jobs` | `aiservices:write:summarizer_batch_job:admin` |
| Translator | `POST /aiservices/translator/translate` | `aiservices:write:translate:admin` |
| Translator | `POST /aiservices/translator/jobs` | `aiservices:write:translator_batch_job:admin` |
| Voice Agent | `GET /aiservices/voice-agent/live` | `aiservices:read:voice-agent:admin` |

Batch read, list, and cancel operations each declare their own `aiservices:read:*:admin` or `aiservices:delete:*:admin` scope. Read them from the catalog instead of assuming one shared batch scope.

## Clinical Notes (documented, not in this catalog)

The AI Services docs also cover a Clinical Notes API:

- `POST /aiservices/clinical_notes/generate` for synchronous file or URL input
- `POST /aiservices/clinical_notes/job` and `GET /aiservices/clinical_notes/job/{job_id}` for asynchronous jobs

It supports `SOAP`, `GENERAL_MEDICINE`, and `GENERAL_BEHAVIORAL_HEALTH` templates. It does not appear in the current `ai-services` OpenAPI catalog, so treat the docs as its source until an operation appears here.
