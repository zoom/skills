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

**Current API audit:** The production V2 WebSocket path and session shape below were verified against the official sample and a live connection on 2026-10-20.

## Routing Guardrail

- Real-time microphone conversation → use this skill.
- One uploaded audio/video file needs transcript plus summary → use [Speech Summarization](../speech-summarization/SKILL.md).
- Live meeting audio/video with meeting metadata → use [RTMS](../rtms/SKILL.md).
- One-shot transcription only → use [Scribe](../scribe/SKILL.md).
- Existing transcript text → use [Summarizer](../summarizer/SKILL.md).
- Typed classification or routing → use [Decisions](../decisions/SKILL.md).

Do not substitute the older OpenAI-style `audio.input` / `audio.output` session payload for the official V2 sample's `listen` / `think` / `speak` payload.

## Production Contract

```text
wss://api.zoom.us/v2/ws/aiservices/voice-agent/live
Sec-WebSocket-Protocol: voice-agent.zoom.v2
x-api-key: $ZOOM_AI_SERVICES_API_KEY
```

Required key scope: `aiservices:read:voice-agent:admin`. The catalog also accepts the `voice-agent.zoom.v1` subprotocol; the official V2 sample uses `voice-agent.zoom.v2`.

The `/v2/ws/aiservices/...` path is the working WebSocket route used by the official sample. Do not use the shorter `/v2/aiservices/voice-agent/live` path merely because the OpenAPI catalog lists it as an upgrade operation; on 2026-10-20 that catalog path returned HTTP `404` for a project whose `/v2/ws/...` route succeeded.

The official sample reads the same credential from `ZOOM_API_KEY`. Use one name in your deployment; `ZOOM_AI_SERVICES_API_KEY` is this skill's preferred name because it distinguishes the API key from Build-platform JWT pairs.

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

The official sample uses `gemma-4`. Treat model availability as deployment-specific and confirm it through `session.updated`; a model that is not configured returns a non-fatal `error` event.

Function tools are declared under `think.tools` as OpenAI-compatible function definitions. Tool results are returned with `conversation.item.create` using `item.type = "function_call_output"`, followed by `response.create`.

## Wire Lifecycle

1. Wait for `session.created`.
2. Send `session.update` and wait for `session.updated` before sending audio.
3. Stream `input_audio_buffer.append` frames containing base64 PCM16 LE mono audio. The official sample sends 20 ms frames at 16 kHz.
4. Consume transcript, response, tool-call, and audio events.
5. On `input_audio_buffer.speech_started`, interrupt queued playback and optionally truncate the active item with `conversation.item.truncate` and `audio_end_ms`.
6. On `response.done`, inspect `response.status`; cancelled, failed, and incomplete responses must not be treated as successful playback.

Important server events include:

- `session.created`, `session.updated`
- `input_audio_buffer.speech_started`
- `conversation.item.input_audio_transcription.delta`
- `conversation.item.input_audio_transcription.completed`
- `response.created`
- `response.output_audio_transcript.delta` / `.done`
- `response.output_audio.delta` / `.done`
- `response.output_item.done`
- `response.function_call_arguments.done`
- `response.done`
- `error`

Audio deltas contain base64 PCM16. Read the output rate from the updated session; the official sample defaults to 16 kHz. Maintain a small playback lead, schedule buffers sequentially, and carry odd bytes across chunk boundaries.

## Failure Handling

- `404` on the upgrade: endpoint or entitlement not provisioned for the project; verify Build Account or Universal Credit access and the `/v2/ws/aiservices/...` path.
- `401` / `403`: invalid key, missing `voice-agent` permission, or relay auth failure.
- `426`: the client did not request a WebSocket upgrade.
- `429`: rate limited; retry with backoff.
- `503`: service unavailable.
- A model-not-configured error is an event-level failure, not necessarily a closed socket.
- Anything before a valid `session.update` is a non-fatal error; the connection may remain open.

## References

- [V2 wire contract and event shapes](references/api-contract.md)
- [Minimal Node.js relay](examples/node-relay.md)
