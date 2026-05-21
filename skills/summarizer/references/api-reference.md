# Zoom AI Services Summarizer API Reference

Canonical sources:
- OpenAPI JSON: https://developers.zoom.us/api-hub/ai-services/methods/endpoints.json
- Docs overview: https://developers.zoom.us/docs/ai-services/summarizer/
- Fast mode docs: https://developers.zoom.us/docs/ai-services/summarizer/fast-mode/
- Batch mode docs: https://developers.zoom.us/docs/ai-services/summarizer/batch-mode/
- Base URL: `https://api.zoom.us/v2`

## Endpoint Inventory

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/aiservices/summarizer/summarize` | Summarize (Synchronous) | `summarize` |
| POST | `/aiservices/summarizer/jobs` | Submit Batch Summarizer Job | `submitBatchSummarizer` |
| GET | `/aiservices/summarizer/jobs` | List Batch Summarizer Jobs | `listBatchSummarizerJobs` |
| GET | `/aiservices/summarizer/jobs/{jobId}` | Get Batch Summarizer Job | `getBatchSummarizerJob` |
| DELETE | `/aiservices/summarizer/jobs/{jobId}` | Cancel Batch Summarizer Job | `cancelBatchSummarizerJob` |
| GET | `/aiservices/summarizer/jobs/{jobId}/files` | List Batch Summarizer Job Files | `listBatchSummarizerJobFiles` |
| GET | `/aiservices/summarizer/jobs/{jobId}/files/{fileId}` | Get Batch Summarizer Job File | `getBatchSummarizerJobFile` |

## Fast Mode Request Shape

`POST /aiservices/summarizer/summarize`

Required top-level fields:
- `input.text`
- `config`

Common config fields:
- `summary_type`: currently `conversation`
- `task`: `recap`, `summary`, `action_items`, `full_summary`
- `language`: BCP-47 output locale code

Optional:
- `reference_id`

Response keys:
- `request_id`
- `task`
- `result.text`
- `model`

## Batch Request Shape

`POST /aiservices/summarizer/jobs`

Required areas:
- `input`
- `output`
- `config`

Input modes:
- `SINGLE`
- `PREFIX`
- `MANIFEST`

Supported transcript input formats:
- `.vtt`
- `.srt`
- `.txt`

Storage/auth fields commonly used:
- `input.source`
- `input.uri`
- `input.manifest`
- `input.filters.include_globs`
- `input.filters.exclude_globs`
- `input.auth.aws.access_key_id`
- `input.auth.aws.secret_access_key`
- `input.auth.aws.session_token`
- `output.destination`
- `output.uri`
- `output.layout`
- `output.auth.aws.*`

Optional:
- `reference_id`
- `notifications.webhook_url`
- `notifications.secret`

## Limits and Constraints

| Area | Value |
|------|-------|
| Fast input size | `96 KB` |
| Prefix job max files | `10,000` |
| Manifest job max files | `1,000` |
| Batch max file size | `96 KB` |
| Batch max characters per job | `100,000,000` |

## Supported Output Languages

If `language` is omitted, output defaults to English.

| Language | Locale code |
|----------|-------------|
| English | `en-US` |
| Chinese (Simplified) | `zh-CN` |
| Japanese | `ja-JP` |
| Spanish | `es-ES` |
| French | `fr-FR` |
| German | `de-DE` |
| Portuguese | `pt-BR` |
| Italian | `it-IT` |
| Arabic (Saudi Arabia) | `ar-SA` |
| Arabic (UAE) | `ar-AE` |
