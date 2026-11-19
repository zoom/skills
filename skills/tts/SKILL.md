---
name: tts
description: "Zoom AI Services Text-to-Speech for one-shot plain-text synthesis. Use for streaming PCM speech, voice selection, speed and volume controls, IVR prompts, notifications, accessibility, and short narration."
triggers:
  - "text to speech"
  - "tts"
  - "synthesize speech"
  - "zoom ai tts"
  - "pcm speech"
  - "ivr prompt audio"
---

# Zoom AI Services Text-to-Speech

Source audit: 2026-11-19 docs snapshot and `openapi-ai-services.json`.

Text-to-Speech converts one complete plain-text input into streamed speech over HTTPS. It is intended for IVR, notifications, accessibility experiences, and short-form narration. The endpoint is:

```text
POST https://api.zoom.us/v2/aiservices/tts/synthesize
```

Required granular scope: `aiservices:write:synthesize:admin`. OpenAPI rate-limit label: `MEDIUM`.

## Request

Send `Content-Type: application/json` with one UTF-8 JSON object. Only `input` is required; all settings are top-level, and there is no `config` object.

| Field | Required | Default | Constraint |
|---|---|---|---|
| `input` | Yes | — | Non-empty plain text, at most 4,000 Unicode code points including spaces and punctuation. Whitespace-only input is rejected; input is never truncated. |
| `voice` | No | `jenna` | One of ten case-insensitive IDs: `alex`, `ariel`, `ava`, `brooke`, `hazel`, `jenna`, `mark`, `reid`, `scott`, `tony`. |
| `speed` | No | `1.0` | `0.5` through `2.0`, inclusive. |
| `volume` | No | `1.0` | Greater than `0` and at most `10`. |
| `response_format` | No | `pcm_44100` | Only `pcm_44100`; the value is case-sensitive. |

Unknown fields, duplicate keys, trailing data, and explicit null values are rejected. The whole request body must be at most 32 KiB (32,768 bytes).

```bash
curl --fail-with-body --no-buffer \
  https://api.zoom.us/v2/aiservices/tts/synthesize \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  --dump-header tts.headers \
  --output tts.pcm \
  -d '{"input":"Hello everyone, welcome to Zoom AI Services.","voice":"jenna","speed":1.0,"volume":1.0,"response_format":"pcm_44100"}'
```

## Audio Stream

A `200` response starts as soon as the first audio is ready. Its body is raw signed 16-bit little-endian mono PCM at 44,100 Hz (88,200 bytes per second), with no WAV/RIFF header, no JSON completion event, and no `Content-Length`. HTTP/1.1 uses chunked transfer encoding.

Reads do not always end on a two-byte sample boundary; carry an incomplete sample into the next read. Start playback as bytes arrive, but treat the result as complete only when the response body ends normally.

## Errors and Recovery

Before streaming begins, errors use JSON with `code`, `reason`, `message`, and `metadata`:

| Status | Reason or meaning | Handling |
|---|---|---|
| `400` | `INVALID_INPUT`, `UNSUPPORTED_VOICE`, `UNSUPPORTED_FORMAT`, or `TEXT_TOO_LONG` | Correct the body, voice, format, or input length. |
| `401` | Authentication failure | Check the API key or JWT and scope. |
| `402` | `QUOTA_EXCEEDED` | Raise the credit limit or wait for reset. |
| `413` | `PAYLOAD_TOO_LARGE` | Reduce the body to 32,768 bytes. |
| `415` | `UNSUPPORTED_MEDIA` | Use `application/json`. |
| `429` | `RATE_LIMIT_EXCEEDED` | Honor `Retry-After`, then retry. |
| `500` | `INTERNAL_ERROR` | Keep the `x-zm-trackingid` for support. |
| `503` | `SERVICE_UNAVAILABLE` | Retry after 10 seconds; honor `Retry-After` and `metadata.retry_after_seconds`. |

After streaming starts, the status cannot change. A mid-stream failure ends the HTTP/1.1 response without its final chunk or resets the HTTP/2 stream. Treat partial audio as incomplete, retry from the beginning with exponential backoff and jitter, and account for audio already played. Closing the connection cancels synthesis; pause/resume is not supported.

Every response includes `x-zm-trackingid`. Keep it, the approximate time, and the final state for support.

## Limits

- US English (`en-US`) input only; there is no `language` field, and an accent does not extend language support.
- Plain text only; SSML tags are spoken rather than interpreted.
- No streaming text input, batch synthesis, model selection, or output formats besides `pcm_44100`.

## Routing

- Uploaded/stored media to text: [Scribe](../scribe/SKILL.md).
- Transcript summaries or action items: [Summarizer](../summarizer/SKILL.md).
- Interactive microphone conversation with spoken responses: [Voice Agent](../voice-agent/SKILL.md).
- General platform routing: [Zoom General](../general/SKILL.md).

## Sources

- Snapshot 2026-11-19: `developers.zoom.us/docs/ai-services/tts.md`
- Snapshot 2026-11-19: `developers.zoom.us/docs/ai-services/tts/fast-mode.md`
- OpenAPI snapshot 2026-11-19: `developers.zoom.us/docs/api/ai-services/openapi-ai-services.json`
