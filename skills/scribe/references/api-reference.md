# Zoom AI Services Scribe API Reference

Canonical sources:
- OpenAPI JSON: https://developers.zoom.us/api/zoap/ai-services/methods/
- Docs overview: https://developers.zoom.us/docs/ai-services/scribe/
- Base URL: `https://api.zoom.us/v2`

## Endpoint Inventory

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/aiservices/scribe/transcribe` | Scribe (Synchronous) | `createFastAsr` |
| GET | `/aiservices/scribe/live` | Live Scribe WebSocket | `liveTranscribe` |
| POST | `/aiservices/scribe/jobs` | Submit Batch Scribe Job | `submitBatchAsr` |
| GET | `/aiservices/scribe/jobs` | List Batch Jobs | `listBatchJobs` |
| GET | `/aiservices/scribe/jobs/{jobId}` | Get Batch Job Status | `getBatchJobStatus` |
| DELETE | `/aiservices/scribe/jobs/{jobId}` | Cancel Batch Job | `cancelBatchJob` |
| GET | `/aiservices/scribe/jobs/{jobId}/files` | List Batch Job Files | `listBatchJobFiles` |
| GET | `/aiservices/scribe/jobs/{jobId}/files/{fileId}` | Get Batch Scribe Job File | `getBatchScribeJobFile` |

## Request Shapes

### `GET /aiservices/scribe/live`

WebSocket URL:

```text
wss://api.zoom.us/v2/aiservices/scribe/live
```

Subprotocol: `live-asr`.

Granular scope: `aiservices:write:live_transcribe:admin`.

Authenticate upstream with `Authorization: Bearer <Zoom AI Services JWT>` per the official docs. A Zoom AI Services JWT, an API key sent as `x-api-key`, and an API key sent as bearer each completed a live handshake on 2026-10-20. A browser client should use a server-side relay.

Initial JSON frame:

```json
{
  "type": "session.update",
  "config": {
    "language": "en-US",
    "interim_results": true,
    "diarization": false,
    "model": "zoom-scribe-multilingual"
  },
  "audio": { "format": "pcm16" }
}
```

`config.language`, `config.model`, `config.interim_results`, `config.diarization`, `config.profanity_filter`, `config.vocabulary`, `config.vocabulary_id`, `outputs.text_polishing`, and `reference_id` are supported. Use either `config.vocabulary` or `config.vocabulary_id`, not both.

Stream signed little-endian 16 kHz mono PCM16 as binary frames of about 100 ms, or `audio.format: "opus"`. Do not add a WAV header or base64-encode binary audio. To attach a source timestamp or participant metadata, send an `input_audio_buffer.append` JSON text frame immediately followed by exactly one binary audio frame.

Handle `session.created`, `session.updated`, `input_audio_buffer.speech_started`, `input_audio_buffer.speech_stopped`, `transcription.delta`, `transcription.completed`, `error`, and `session.closed`. Send `{"type":"session.close"}` to stop and keep reading until `session.closed` arrives.

Session limits: 120 minutes maximum and a 30 second idle timeout without audio events.

### `POST /aiservices/scribe/transcribe`

Required top-level fields:
- `file`
- `config`

Common config fields:
- `language`
- `word_time_offsets`
- `channel_separation`
- `timestamps`
- `output_format`
- `profanity_filter`
- `diarization`

Response keys:
- `request_id`
- `duration_sec`
- `model`
- `result`

### `POST /aiservices/scribe/jobs`

Required top-level fields:
- `input`
- `output`
- `config`

Input subfields:
- `mode` (`SINGLE`, `PREFIX`, `MANIFEST`)
- `source` (`S3` in current spec)
- `uri`
- `manifest`
- `filters.include_globs`
- `filters.exclude_globs`
- `auth.aws.access_key_id`
- `auth.aws.secret_access_key`
- `auth.aws.session_token`

Output subfields:
- `destination`
- `uri`
- `layout` (`SINGLE`, `PREFIX`, `ADJACENT`)
- `auth.aws.*`

Config subfields:
- `language`
- `word_time_offsets`
- `channel_separation`
- `diarization`
- `profanity_filter`
- `output_format`
- `segmentation_mode`

Optional:
- `reference_id`
- `notifications.webhook_url`
- `notifications.secret`

Response keys:
- `job_id`
- `state`
- `submitted_at`

### `GET /aiservices/scribe/jobs`

Query params:
- `state`
- `page_size`
- `next_page_token`

Response keys:
- `jobs`
- `next_page_token`

### `GET /aiservices/scribe/jobs/{jobId}`

Path params:
- `jobId`

Response keys:
- `job_id`
- `state`
- `submitted_at`
- `summary`

### `GET /aiservices/scribe/jobs/{jobId}/files`

Path params:
- `jobId`

Query params:
- `page_size`
- `next_page_token`

Response keys:
- `files`
- `next_page_token`

### `GET /aiservices/scribe/jobs/{jobId}/files/{fileId}`

Path params:
- `jobId`
- `fileId`

Response keys:
- `file_id`
- `input_uri`
- `output_uri`
- `state`
- `duration_sec`
- `error`

## Current Limits and Constraints Observed in Sources

- Batch manifest max: `1000` file URIs.
- `include_globs` max items: `10`.
- `exclude_globs` max items: `10`.
- Audio/media formats called out in docs: `WAV`, `MP3`, `M4A`, `MP4`.
- Batch job rate limit label in the OpenAPI description: `LIGHT`.
