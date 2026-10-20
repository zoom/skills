# Voice Agent V2 wire contract

Verified against the official Voice Agent sample and a live `voice-agent.zoom.v2` connection on 2026-10-20.

## Endpoint and authentication

```text
wss://api.zoom.us/v2/ws/aiservices/voice-agent/live
Sec-WebSocket-Protocol: voice-agent.zoom.v2
x-api-key: ZOOM_AI_SERVICES_API_KEY
```

The API reference also accepts bearer authentication. The browser cannot set either header on a native `WebSocket`, so terminate the browser connection at a trusted server relay and authenticate the upstream socket there.

## Catalog route versus V2 route

The public catalog lists `GET /aiservices/voice-agent/live` with granular scope `aiservices:read:voice-agent:admin`. Its prose describes the session shape with `model`, `instructions`, `output_modalities`, and `audio.input.format`, and claims the socket uses no subprotocol. That prose contradicts the operation's own required `Sec-WebSocket-Protocol` header, whose enum is `voice-agent.zoom.v1` or `voice-agent.zoom.v2`.

The official V2 sample connects to the `/v2/ws/aiservices/voice-agent/live` route with `voice-agent.zoom.v2` and the `listen` / `think` / `speak` session body shown below. That combination was verified live.

Treat these as two surfaces of the same capability:

- `/v2/ws/aiservices/voice-agent/live` + `voice-agent.zoom.v2` — verified working route; returned `session.updated` with `think.provider = zoom/gemma-4` on 2026-10-20. Use this when following the official sample.
- `/v2/aiservices/voice-agent/live` — catalog operation; declares `voice-agent.zoom.v1` or `voice-agent.zoom.v2` as required subprotocols while its prose describes OpenAI-style session fields. On 2026-10-20 this path returned HTTP `404` for a project whose `/v2/ws/...` route succeeded, so do not switch paths to fix an auth error.

Do not mix the two session bodies on one socket. If a connection is rejected or a session update is refused, confirm which route and session shape the client actually sent before changing audio code.

## Session update

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

Wait for `session.updated` before sending microphone frames.

## Client frames

Audio input:

```json
{
  "type": "input_audio_buffer.append",
  "audio": "<base64 PCM16 LE mono>"
}
```

Interruption:

```json
{
  "type": "conversation.item.truncate",
  "item_id": "<active assistant item>",
  "content_index": 0,
  "audio_end_ms": 1250
}
```

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

The V2 sample returns flat JSON events, not the older `{type,payload}` wrapper used by some preview clients. Handle both defensively if supporting both surfaces.

Relevant event names:

- `session.created`
- `session.updated`
- `input_audio_buffer.speech_started`
- `conversation.item.input_audio_transcription.delta`
- `conversation.item.input_audio_transcription.completed`
- `conversation.item.input_audio_transcription.failed`
- `response.created`
- `response.output_audio_transcript.delta`
- `response.output_audio_transcript.done`
- `response.output_audio.delta`
- `response.output_audio.done`
- `response.output_item.done`
- `response.function_call_arguments.done`
- `response.done`
- `error`

`response.output_audio.delta.delta` is base64 PCM16. Use the output rate from `session.updated` and do not assume a fixed rate if the server supplies one.

## Interruption and playback

On speech start, stop queued playback for the active item and clear scheduled sources. If truncating, compute `audio_end_ms` from the portion already played. Keep a small playback lead so chunks remain contiguous, and reject playback buffers that grow beyond a bounded duration.

## Tool-call handling

Accept function calls only from `response.output_item.done` or `response.function_call_arguments.done`. De-duplicate by `call_id`, execute each call once, append the function result, and call `response.create` after all pending tool calls complete. Discard results from a cancelled or superseded generation.

## Failure states

- HTTP 404 on upgrade: route or project entitlement not provisioned.
- HTTP 401/403: key or `voice-agent` permission rejected.
- HTTP 426: WebSocket upgrade required.
- HTTP 429: rate limited.
- HTTP 503: service unavailable.
- Model-not-configured and invalid-session events are reported as `error` frames and may leave the socket open.
