---
name: voice-agent
description: "Zoom AI Services Voice Agent V2 for real-time microphone conversations over WebSocket, including VAD, PCM audio, speech-to-speech or text responses, interruption handling, and function tools. Use for interactive voice agents; not one-shot media summarization or live meeting transcription."
triggers:
  - "voice agent"
  - "realtime voice agent"
  - "speech to speech"
  - "voice-agent.zoom.v2"
  - "live microphone agent"
  - "voice agent websocket"
  - "barge in voice agent"
---

# Zoom AI Services Voice Agent

Use Voice Agent for an interactive microphone conversation where the service transcribes speech, runs an AI turn, and streams spoken or text output in real time.

**Current API audit:** The V2 route, session shape, event stream, and error recovery below were verified against the official sample and live connections on 2026-10-20. The documented V1 surface was verified on the same `/v2/ws/` route with a different subprotocol and body.

## Routing Guardrail

- Real-time microphone conversation → use this skill.
- One uploaded audio/video file needs transcript plus summary → use [Speech Summarization](../speech-summarization/SKILL.md).
- Live meeting audio/video with meeting metadata → use [RTMS](../rtms/SKILL.md).
- One-shot transcription only → use [Scribe](../scribe/SKILL.md).
- Existing transcript text → use [Summarizer](../summarizer/SKILL.md).
- Typed classification or routing → use [Decisions](../decisions/SKILL.md).

Match the session body to the subprotocol: V2 uses `listen` / `think` / `speak`, while the documented V1 surface uses OpenAI-style `audio.input` / `audio.output`. A mismatch returns `error.code = shape_switch_not_allowed`.

## Production Contract

```text
wss://api.zoom.us/v2/ws/aiservices/voice-agent/live
Sec-WebSocket-Protocol: voice-agent.zoom.v2
x-api-key: $ZOOM_AI_SERVICES_API_KEY
```

Required key scope: `aiservices:read:voice-agent:admin`.

Two subprotocols are live on that route, and they are not interchangeable:

| Surface | Subprotocol | Session body | Server events |
|---|---|---|---|
| V2, official sample | `voice-agent.zoom.v2` | `listen` / `think` / `speak` | flat JSON events |
| V1, docs live-mode page | `voice-agent.zoom.v1` | OpenAI-style `audio.input` / `audio.output` | `{type, payload}` envelope |

The documented upgrade path `wss://api.zoom.us/v2/aiservices/voice-agent/live` (without `/ws/`) returned HTTP `404` on 2026-10-20 for every subprotocol tried, including `voice-agent.zoom.v1`. The OpenAPI catalog lists `GET /aiservices/voice-agent/live`, but both working sockets are under `/v2/ws/aiservices/...`.

V1 accepts `Authorization: Bearer <API key or Zoom AI Services JWT>`. V2 was verified with `x-api-key` and also accepts bearer auth. The official sample reads the credential from `ZOOM_API_KEY`; this skill uses `ZOOM_AI_SERVICES_API_KEY` to distinguish it from Build-platform JWT pairs. Use one name in your deployment.

Keep the key server-side. A browser `WebSocket` cannot set `x-api-key`, so use a server-side relay:

```text
browser microphone WebSocket -> trusted /live relay -> api.zoom.us Voice Agent
```

The relay must validate `Origin` and `Host`, request `voice-agent.zoom.v2`, inject the key upstream, enforce frame-size and backpressure limits, and never expose the key to the browser.

## Session Shape

The first upstream client frame must be `session.update`:

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
      "instructions": "You are an AI assistant interacting with a user through spoken conversation."
    },
    "speak": {
      "format": { "type": "pcm16" },
      "voice": "ariel"
    }
  }
}
```

The official sample uses `gemma-4`. Treat model availability as deployment-specific and confirm it through `session.updated`; an unconfigured model returns a non-fatal `error` event and the socket stays open.

`speak` is optional; omitting it defaults to voice `jenna` at 16 kHz. Valid voice IDs are `alex`, `ariel`, `ava`, `brooke`, `hazel`, `jenna`, `mark`, `reid`, `scott`, and `tony`. Input transcription accepts only `en`; `ja` and `en-US` were both rejected on 2026-10-20.

Function tools are declared under `think.tools` as OpenAI-compatible function definitions. Tool results are returned with `conversation.item.create` using `item.type = "function_call_output"`, followed by `response.create`.

## Wire Lifecycle

1. Wait for `session.created`.
2. Send `session.update` and wait for `session.updated` before sending audio.
3. Stream `input_audio_buffer.append` frames containing base64 PCM16 LE mono audio. The official sample sends 20 ms frames at 16 kHz.
4. Consume transcript, response, tool-call, and audio events.
5. On `input_audio_buffer.speech_started`, stop local playback, send `response.cancel` with the active `response.id`, then send `conversation.item.truncate` with `item_id` and `audio_end_ms`. Verified: cancel produces `response.done` with status `cancelled`.
6. On `response.done`, inspect `response.status`; cancelled, failed, and incomplete responses must not be treated as successful playback.
7. A bare `response.create` with no audio produces a spoken turn, which is useful for a connectivity check.

Important server events include:

- `session.created`, `session.updated`
- `input_audio_buffer.speech_started`
- `conversation.item.input_audio_transcription.delta`
- `conversation.item.input_audio_transcription.completed`
- `response.created`
- `response.output_item.added`
- `conversation.item.added`
- `response.content_part.added` / `.done`
- `response.output_audio_transcript.delta` / `.done`
- `response.output_audio.delta` / `.done`
- `response.output_item.done`
- `conversation.item.done`
- `response.function_call_arguments.done`
- `response.done`
- `error`

V1 emits the same concepts inside a `{type, payload}` envelope and adds `input_audio_buffer.speech_stopped`, `input_audio_buffer.timeout_triggered`, and `response.output_text.delta` for text-only sessions.

Audio deltas contain base64 PCM16. Read the output rate from the updated session; the official sample defaults to 16 kHz. Maintain a small playback lead, schedule buffers sequentially, and carry odd bytes across chunk boundaries.

## Failure Handling

- `404` on the upgrade: wrong path or missing entitlement. Both surfaces use `/v2/ws/aiservices/...`; the documented `/v2/aiservices/...` path returns 404.
- `401` / `403`: invalid key, missing `aiservices:read:voice-agent:admin`, or relay auth failure.
- `426`: the client did not request a WebSocket upgrade.
- `429`: rate limited; retry with backoff.
- `503`: service unavailable.
- `error` with `invalid_request_error`: model, voice, language, format, or first-frame violation. The socket stays open; correct the setting and resend `session.update`.
- `error.code = shape_switch_not_allowed`: the session body does not match the negotiated subprotocol.
- Audio before `session.update` returns `first event must be a session.update carrying a realtime session`; the socket stays open.

## References

- [V2 wire contract and event shapes](references/api-contract.md)
- [Minimal Node.js relay](examples/node-relay.md)

## Sources

- Docs live mode: https://developers.zoom.us/docs/ai-services/voice-agent/live-mode/
- Catalog: https://developers.zoom.us/api/zoap/ai-services/methods/
