# Speech Summarization API contract

Verified against the public API reference and official Node.js sample on 2026-10-20.

## Endpoint

```http
POST https://api.zoom.us/v2/aiservices/speech-summarization/summarize
```

Authentication:

```http
x-api-key: ZOOM_AI_SERVICES_API_KEY
```

or:

```http
Authorization: Bearer ZOOM_AI_SERVICES_API_KEY
```

Required scope: `aiservices:write:transcribe-summarize:admin`.

## Request

Use multipart form data with exactly the fields the service consumes:

```js
const form = new FormData();
form.append('config', JSON.stringify(config));
form.append('file', new Blob([await readFile(audioPath)], {
  type: 'application/octet-stream',
}), basename(audioPath));
```

Use the official configuration shape:

```json
{
  "transcribe": {
    "language": "en-US",
    "word_time_offsets": false,
    "channel_separation": false,
    "diarization": true,
    "profanity_filter": false,
    "output_format": "json"
  },
  "summary": {
    "language": "en-US",
    "citations": true,
    "should_remove_sentiment": false,
    "pii_redaction": false,
    "enable_dynamic_template": true
  }
}
```

The official sample does not send `model` or `summary_length`. Do not add them merely because the playground exposes equivalent controls unless the target API contract requires them.

## SSE events

```text
event: transcribe_started
data: {"status":"transcribing"}
```

```text
event: transcribe_complete
data: {"transcript":{...},"latency_ms":5234,"body_bytes":12456,"status_code":200}
```

`transcript` may be a JSON string or object. If it is a string, parse it before reading `result`, `text_display`, `transcript`, `text`, or `segments`.

```text
event: summary_delta
data: {"summary":"## Meeting Summary\n...","metadata":{}}
```

Each summary frame contains the complete accumulated summary. Print only `nextText.slice(previousText.length)`.

```text
event: done
data: {"status":"complete","total_latency_ms":8500,"transcribe_ms":5234,"summary_ms":3266}
```

```text
event: error
data: {"error":"error message","stage":"transcribe|summary|summary_preparation"}
```

## Summary extraction fallbacks

```js
function readSummary(payload) {
  const result = payload.summary_result;
  if (typeof result === 'string' && result) return result;

  const overall = result?.summarization?.overall_summary;
  if (typeof overall === 'string' && overall) return overall;

  if (payload.metadata) {
    const nested = readSummary(payload.metadata);
    if (nested) return nested;
  }

  return payload.summary || payload.text;
}
```

## Limits and failure states

- Maximum file size: 50 MB
- Maximum duration: 2 hours
- Supported audio: WAV, MP3, M4A, FLAC, OGG, WebM
- Supported video: MP4, MOV
- Minimum sample rate: 8 kHz
- Recommended sample rate: 16 kHz or higher
- `402 QUOTA_EXCEEDED` means the project lacks credits, not that the payload is invalid
- `502` means an upstream transcription or summary service failed
- `503` means the summary API is not configured
