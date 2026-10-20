# Zoom AI Services API

Authoritative OpenAPI endpoint inventory for Scribe, Summarizer, and Translator, plus a separately identified Decisions surface that is not yet represented in that inventory.

## Canonical Source

- OpenAPI JSON: https://developers.zoom.us/api/zoap/ai-services/methods/
- Base URL: `https://api.zoom.us/v2`
- Product skills: [../../scribe/SKILL.md](../../scribe/SKILL.md), [../../summarizer/SKILL.md](../../summarizer/SKILL.md), [../../translator/SKILL.md](../../translator/SKILL.md), [../../decisions/SKILL.md](../../decisions/SKILL.md)
- Authentication details: [authentication.md](authentication.md)

## Notes

- Current OpenAPI surface includes Scribe transcription, Summarizer transcript summarization, and Translator text translation.
- Decisions was live-tested separately at `POST /aiservices/decisions/decide` on 2026-10-17. Do not count it as part of the OpenAPI coverage metrics below until the canonical document includes it; use the [Decisions skill](../../decisions/SKILL.md) for its observed contract and workarounds.
- Auth uses Build-platform JWT, which differs from the standard OAuth-centric paths in most REST API product areas.
- Use this file for endpoint discovery and inventory. Use the product skills for workflow, webhook, and mode-selection guidance.

## Coverage

| Metric | Value |
|--------|-------|
| Endpoint operations | 21 |
| Path templates | 15 |
| Tags | 3 |

## Tag Index

| Tag | Operations |
|-----|------------|
| Scribe | 7 |
| Summarizer | 7 |
| Translator | 7 |

## Endpoints by Tag

### Scribe

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/aiservices/scribe/jobs` | List Batch Jobs | `listBatchJobs` |
| POST | `/aiservices/scribe/jobs` | Submit Batch Scribe Job | `submitBatchAsr` |
| GET | `/aiservices/scribe/jobs/{jobId}` | Get Batch Job Status | `getBatchJobStatus` |
| DELETE | `/aiservices/scribe/jobs/{jobId}` | Cancel Batch Job | `cancelBatchJob` |
| GET | `/aiservices/scribe/jobs/{jobId}/files` | List Batch Job Files | `listBatchJobFiles` |
| GET | `/aiservices/scribe/jobs/{jobId}/files/{fileId}` | Get Batch Scribe Job File | `getBatchScribeJobFile` |
| POST | `/aiservices/scribe/transcribe` | Scribe (Synchronous) | `createFastAsr` |

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

## Separately audited Decisions surface

| Method | Endpoint | Summary | Status |
|--------|----------|---------|--------|
| POST | `/aiservices/decisions/decide` | Typed predicate, choice, score, and image decisions | Live-tested production behavior; not present in the current OpenAPI inventory |

Production Decisions authentication was observed using `x-api-key` from a Zoom Build Platform project. Build Platform was previously known as the Video SDK account. The project can be associated with a CPaaS account under a Zoom Meetings account, a standalone Build Platform account through `marketplace.zoom.us`, or a CPaaS account through `aiservice.zoom.us`. This differs from the Build-platform JWT guidance recorded for the OpenAPI-listed services. Keep credentials server-side and read the [Decisions skill](../../decisions/SKILL.md) before implementing model selection, request validation, retries, Boolean normalization, or score-label mapping.
