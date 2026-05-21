# Zoom AI Services Translator API Reference

Canonical sources:
- OpenAPI JSON: https://developers.zoom.us/api-hub/ai-services/methods/endpoints.json
- Docs overview: https://developers.zoom.us/docs/ai-services/translator/
- Fast mode docs: https://developers.zoom.us/docs/ai-services/translator/fast-mode/
- Batch mode docs: https://developers.zoom.us/docs/ai-services/translator/batch-mode/
- Base URL: `https://api.zoom.us/v2`

## Endpoint Inventory

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/aiservices/translator/translate` | Translate (Synchronous) | `translate` |
| POST | `/aiservices/translator/jobs` | Submit Batch Translator Job | `submitBatchTranslator` |
| GET | `/aiservices/translator/jobs` | List Batch Translator Jobs | `listBatchTranslatorJobs` |
| GET | `/aiservices/translator/jobs/{jobId}` | Get Batch Translator Job | `getBatchTranslatorJob` |
| DELETE | `/aiservices/translator/jobs/{jobId}` | Cancel Batch Translator Job | `cancelBatchTranslatorJob` |
| GET | `/aiservices/translator/jobs/{jobId}/files` | List Batch Translator Job Files | `listBatchTranslatorJobFiles` |
| GET | `/aiservices/translator/jobs/{jobId}/files/{fileId}` | Get Batch Translator Job File | `getBatchTranslatorJobFile` |

## Fast Mode Request Shape

`POST /aiservices/translator/translate`

Required top-level fields:
- `text`
- `config.source_language`
- `config.target_languages`

Constraints:
- `text` max: `4,000` characters
- `target_languages` must contain exactly one locale
- either source or target must be `en-US`
- non-English to non-English pairs are not supported yet

Response keys:
- `request_id`
- `result.translations`

## Batch Request Shape

`POST /aiservices/translator/jobs`

Required areas:
- `input`
- `output`
- `config.source_language`
- `config.target_languages`

Input modes:
- `SINGLE`
- `PREFIX`
- `MANIFEST`

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
- `output.overwrite`
- `output.auth.aws.*`

Output layouts:
- `ADJACENT`
- `PREFIX`

Optional:
- `reference_id`
- `notifications.webhook_url`
- `notifications.secret`

## Limits and Constraints

| Area | Value |
|------|-------|
| Fast input size | `4,000` characters |
| Batch max characters per file | `4,000` |
| Batch max file size | `16 KB` |
| Manifest job max files | `1,000` |
| Batch max characters per job | `100,000,000` |
| Batch max characters per day | `1,000,000,000` |

## Supported Languages

Translation is bidirectional with English. Every request must have either `source_language` or `target_languages` set to `en-US`.

| Locale code | Language |
|-------------|----------|
| `en-US` | English |
| `zh-CN` | Chinese (Simplified) |
| `zh-TW` | Chinese (Traditional) |
| `ja-JP` | Japanese |
| `ko-KR` | Korean |
| `es-ES` | Spanish |
| `fr-FR` | French |
| `de-DE` | German |
| `pt-BR` | Portuguese |
| `it-IT` | Italian |
