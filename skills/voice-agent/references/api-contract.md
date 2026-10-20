# Voice Agent V2 wire contract

Verified against the official Voice Agent sample and a live `voice-agent.zoom.v2` connection on 2026-10-20.

## Endpoint and authentication

```text
wss://api.zoom.us/v2/ws/aiservices/voice-agent/live
Sec-WebSocket-Protocol: voice-agent.zoom.v2
x-api-key: ZOOM_AI_SERVICES_API_KEY
```

Bearer authentication is also accepted; both an API key and a Zoom AI Services JWT completed handshakes on 2026-10-20. The browser cannot set either header on a native `WebSocket`, so terminate the browser connection at a trusted server relay and authenticate the upstream socket there.

## Supported surfaces and routes

The catalog lists `GET /aiservices/voice-agent/live` with scope `aiservices:read:voice-agent:admin` and a required `Sec-WebSocket-Protocol` header whose enum is `voice-agent.zoom.v1` or `voice-agent.zoom.v2`. Its prose claims the socket has no subprotocol, which contradicts its own parameter.

Live testing on 2026-10-20 produced this matrix:

| Route | Subprotocol | Result |
|---|---|---|
| `/v2/ws/aiservices/voice-agent/live` | `voice-agent.zoom.v2` | works; flat events; V2 session body |
| `/v2/ws/aiservices/voice-agent/live` | `voice-agent.zoom.v1` | works; `{type, payload}` envelope; V1 session body |
| `/v2/aiservices/voice-agent/live` | `v1`, `v2`, or omitted | HTTP 404 |

The docs' connection snippet has the wrong path, not the wrong subprotocol. Both surfaces live under `/v2/ws/aiservices/...`.

A body that does not match the negotiated subprotocol is rejected with `error.type = invalid_request_error` and `error.code = shape_switch_not_allowed`, for example `session shape is locked to 'v2'; cannot switch to 'v1'`.

## V1 session body

The docs live-mode page documents this shape. It works on the `/v2/ws/` route with `voice-agent.zoom.v1`:

```json
{
  "type": "session.update",
  "session": {
    "type": "realtime",
    "model": "gemma-4",
    "output_modalities": ["audio"],
    "instructions": "You are a concise support agent.",
    "audio": {
      "input": {
        "format": { "type": "audio/pcm", "rate": 16000 },
        "transcription": { "model": "zoom_speech", "language": "en" }
      },
      "output": {
        "format": { "type": "audio/pcm", "rate": 16000 },
        "voice": "jenna"
      }
    }
  }
}
```

V1 specifics verified live:

- `output_modalities` accepts `["audio"]` or `["text"]`; the modes cannot be combined.
- Input transcription accepts only `en`. `ja` returns `language 'ja' is not supported; supported language families: en`; `en-US` returns `unsupported transcription language: 'en-US'`.
- Events arrive as `{"type": "...", "payload": {...}}`; read the event from `payload`.
- Default turn detection is `server_vad` with `create_response` and `interrupt_response` enabled.
- V1 adds server events `input_audio_buffer.speech_stopped`, `input_audio_buffer.timeout_triggered`, and `response.output_text.delta`, plus the client events `input_audio_buffer.commit` and `input_audio_buffer.clear`.

## V2 session body

The official sample uses this shape with `voice-agent.zoom.v2`. Events are flat and have no `payload` wrapper.

## V2 session update

The first upstream text frame must be `session.update`:

```json
{
  "type": "session.update",
  "session": {
    "type": "realtime",
    "listen": {
      "transcription": { "language": "en" },
      "format": { "type": "pcm16", "rate": 16000 }
    },
    "think": {
      "provider": { "type": "zoom", "model": "gemma-4" },
      "instructions": "Respond naturally and conversationally.",
      "tools": [
        {
          "type": "function",
          "name": "get_time_in_timezone",
          "description": "Get the current time in an IANA timezone.",
          "parameters": {
            "type": "object",
            "properties": {
              "timezone": { "type": "string" }
            },
            "required": ["timezone"]
          }
        }
      ]
    },
    "speak": {
      "format": { "type": "pcm16" },
      "voice": "ariel"
    }
  }
}
```

Wait for `session.updated` before sending microphone frames. `speak` is optional and defaults to `jenna` at 16 kHz. Valid voice IDs are `alex`, `ariel`, `ava`, `brooke`, `hazel`, `jenna`, `mark`, `reid`, `scott`, and `tony`. Input transcription accepts only `en`; `ja` and `en-US` are rejected.

`response.create` may be sent with no audio to generate a spoken turn. That is a cheap connectivity check, and it was verified to produce a full `response.output_audio.*` stream ending in `response.done` with status `completed`.

## Client frames

Audio input:

```json
{
  "type": "input_audio_buffer.append",
  "audio": "<base64 PCM16 LE mono>"
}
```

Interruption, in order:

```json
{
  "type": "response.cancel",
  "response_id": "<active response id>"
}
```

```json
{
  "type": "conversation.item.truncate",
  "item_id": "<active assistant item>",
  "content_index": 0,
  "audio_end_ms": 1250
}
```

Verified: `response.cancel` with the `response.id` from `response.created` is accepted and the turn ends with `response.done` and status `cancelled`.

Tool output:

```json
{
  "type": "conversation.item.create",
  "item": {
    "type": "function_call_output",
    "call_id": "<call id>",
    "output": "<tool result>"
  }
}
```

Then request the next spoken turn:

```json
{ "type": "response.create" }
```

## Server events

V2 returns flat JSON events. V1 returns the same concepts inside a `{"type": "...", "payload": {...}}` envelope, so read the event from `payload` on V1. Do not assume one shape for both surfaces.

Relevant event names:

- `session.created`
- `session.updated`
- `input_audio_buffer.speech_started`
- `conversation.item.input_audio_transcription.delta`
- `conversation.item.input_audio_transcription.completed`
- `conversation.item.input_audio_transcription.failed`
- `response.created`
- `response.output_item.added`
- `conversation.item.added`
- `response.content_part.added`
- `response.output_audio_transcript.delta`
- `response.output_audio_transcript.done`
- `response.output_audio.delta`
- `response.output_audio.done`
- `response.content_part.done`
- `response.output_item.done`
- `conversation.item.done`
- `response.function_call_arguments.done`
- `response.done`
- `error`

`response.output_audio.delta.delta` is base64 PCM16. Use the output rate from `session.updated` and do not assume a fixed rate if the server supplies one.

## Interruption and playback

On speech start, stop queued playback for the active item and clear scheduled sources. Cancel the in-flight generation with `response.cancel`, then truncate the played item with `audio_end_ms` computed from the portion already heard. Keep a small playback lead so chunks remain contiguous, and reject playback buffers that grow beyond a bounded duration.

## Tool-call handling

Accept function calls only from `response.output_item.done` or `response.function_call_arguments.done`. De-duplicate by `call_id`, execute each call once, append the function result, and call `response.create` after all pending tool calls complete. Discard results from a cancelled or superseded generation.

## Failure states

- HTTP 404 on upgrade: wrong path or missing entitlement. Both surfaces use `/v2/ws/aiservices/...`; the documented `/v2/aiservices/...` path returns 404.
- HTTP 401/403: key or `aiservices:read:voice-agent:admin` rejected.
- HTTP 426: WebSocket upgrade required.
- HTTP 429: rate limited.
- HTTP 503: service unavailable.
- Unconfigured model, unknown voice, unsupported language, unsupported format, and a missing first `session.update` are reported as `invalid_request_error` events. The socket stays open and accepts a corrected `session.update`.
- Mismatched body and subprotocol returns `shape_switch_not_allowed`.
- An active-response conflict is nonfatal; wait for `response.done` before sending `response.create` again.
