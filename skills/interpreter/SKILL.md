---
name: interpreter
description: "Zoom AI Services Interpreter for live speech-to-speech translation over WebSocket. Use for streaming source audio, translated text and PCM audio, optional source transcripts, meetings, live events, classrooms, and support."
triggers:
  - "interpreter"
  - "live translation"
  - "speech translation"
  - "real-time translator"
  - "translate live audio"
  - "interpreter websocket"
---

# Zoom AI Services Interpreter

Source audit: 2026-11-19 docs snapshot and `openapi-ai-services.json`.

Interpreter turns one live source-language audio stream into one target language. It always returns translated text and synthesized audio; source transcription is optional. Final text and audio deltas are never revised.

```text
wss://api.zoom.us/v2/ws/aiservices/interpreter/live
Sec-WebSocket-Protocol: interpreter.zoom.v1
```

Required granular scope: `aiservices:read:interpreterlive:admin`. OpenAPI rate-limit label: `MEDIUM`. Not supported in the Gov cluster.

Authenticate in a trusted backend with `Authorization: Bearer <JWT>` or `x-api-key`. Browser WebSockets cannot set these headers reliably, so relay through a server. Connection rejections include `401` invalid credentials, `402 QUOTA_EXCEEDED`, and `503 SERVICE_DISABLED` or `CAPACITY_EXCEEDED`.

## Session Rules

1. Send `session.update` first. Waiting for `session.created` is allowed but not required.
2. Wait for `session.updated` before audio or `session.close`. Sending either earlier ends the session with fatal `invalid_message`.
3. Stream continuously, including silence.
4. To finish, send `session.close` once and keep reading until `session.closed`.

The configuration is immutable. A later `session.update` produces nonfatal `invalid_request` with `param: session`; open a new connection to change languages or voice. Reconnection always starts a new session; there is no resume.

```json
{
  "type": "session.update",
  "session": {
    "audio": {
      "input": {
        "language": "en-US",
        "transcription": {}
      },
      "output": {
        "language": "es-ES",
        "voice": "sofia"
      }
    }
  }
}
```

All three language/voice fields are required and have no defaults. Source and target must differ. Set `input.transcription` to `{}` to receive source text; omit it or use `null` to disable it. The object has no fields.

## Languages and Voices

The supported source and target tags are `en-US`, `ar-AE`, `de-DE`, `es-ES`, `fr-FR`, `it-IT`, `ja-JP`, `pt-PT`, and `zh-CN`. Tags and voice IDs are case-insensitive; `session.updated` normalizes tags and lowercases voices.

| Target | Voices |
|---|---|
| `en-US` | `sarah`, `cole` |
| `ar-AE` | `layla`, `omar` |
| `de-DE` | `helena`, `felix` |
| `es-ES` | `sofia`, `carlos` |
| `fr-FR` | `camille`, `marc` |
| `it-IT` | `elena`, `luca` |
| `ja-JP` | `hana`, `kenji` |
| `pt-PT` | `beatriz`, `tiago` |
| `zh-CN` | `lin`, `bo` |

Automatic source-language detection and multiple target languages per connection are unsupported.

## Audio Contract

Input and output are signed 16-bit little-endian PCM, 16 kHz, mono, with no WAV/RIFF header. The service does not resample input.

Send audio as Base64 in `session.input_audio_buffer.append`. Send 100-200 ms per event (3,200-6,400 bytes before encoding); 500 ms or 16,000 bytes is the maximum, in complete 16-bit samples. A larger audio event is fatal `invalid_message`; a WebSocket message over 1 MiB closes with `1009`. The server does not acknowledge audio and does not slow down for slow clients.

Output audio arrives in `session.output_audio.delta` as Base64 PCM with `format: pcm_s16le`, `sample_rate: 16000`, and `channels: 1`. Decode and play deltas in order.

## Events

Handle at least:

| Event | Meaning |
|---|---|
| `session.created` | First server event; contains `session.id`, type `translation`, expiry, and defaults. |
| `session.updated` | Configuration accepted; audio may begin. |
| `session.input_transcript.delta` | Final source-text fragment, if enabled. |
| `session.output_transcript.delta` | Final translated-text fragment. |
| `session.output_audio.delta` | Base64 translated PCM chunk. |
| `error` | Contains `error.code`, `fatal`, `retryable`, and often `param`. |
| `session.closed` | Last event; inspect `reason`. |

Each text stream is ordered, but source text, translated text, and audio streams are not synchronized and do not map one-to-one. Append each text stream independently; deltas already include required spacing. Do not block the receive handler while processing audio.

`session.closed.reason: completed` means all output was delivered. Any other reason is an early termination. Fatal errors and network failures can close without `session.closed`, so a normal socket close alone does not prove completion.

## Errors

Use `error.code`, not the human-readable message. Only `session.update` failures can be nonfatal; correct `error.param` and resend the update when `fatal` is false. When `fatal` is true, the server closes and sends no further events.

| Code | Handling |
|---|---|
| `invalid_message` | Fix JSON, fields, event order, or audio. Fatal except for a correctable `session.update`. |
| `invalid_request` | Fix the language, voice, or pair named by `param`; resend a valid update if nonfatal. |
| `policy_violation` | Use a setting the account permits or enable it. |
| `provider_unhealthy` / `provider_timeout` | Reconnect with backoff when `retryable` is true. |
| `session_expired` / `session_limit` / `idle_timeout` | Start a new session. |
| `auth_unavailable` / `internal_error` | Retry with backoff when `retryable`; otherwise contact support. |

The idle limit is 30 seconds without a client event by default; server output does not reset it. Maximum duration is 120 minutes, and accounts may be lower. Resending audio after a retryable failure can duplicate translated output. For support, keep the session ID, configuration, and errors, but never log credentials.

## Routing

- Plain text or stored text files: [Translator](../translator/SKILL.md).
- Transcript only, without translation: [Scribe](../scribe/SKILL.md).
- Interactive agent conversation: [Voice Agent](../voice-agent/SKILL.md).
- One-shot plain-text synthesis: [Text-to-Speech](../tts/SKILL.md).

## Sources

- Snapshot 2026-11-19: `developers.zoom.us/docs/ai-services/interpreter.md`
- Snapshot 2026-11-19: `developers.zoom.us/docs/ai-services/interpreter/live-mode.md`
- OpenAPI snapshot 2026-11-19: `developers.zoom.us/docs/api/ai-services/openapi-ai-services.json`
