# Voice Agent: Real-Time Conversation

Use this workflow for a live microphone conversation where Zoom transcribes speech, runs an AI turn, and streams spoken or text output.

## Skill Chain

- V2 WebSocket contract, session shape, and event handling: [Voice Agent](../../voice-agent/SKILL.md)
- Minimal Node relay: [Voice Agent relay example](../../voice-agent/examples/node-relay.md)
- One-shot media transcript plus summary: [Speech Summarization](../../speech-summarization/SKILL.md)
- Meeting media streams: [RTMS](../../rtms/SKILL.md)
- Typed decisions after a turn: [Decisions](../../decisions/SKILL.md)

## Architecture

```text
browser microphone and playback
  -> trusted /live WebSocket relay
  -> wss://api.zoom.us/v2/ws/aiservices/voice-agent/live
```

The browser cannot attach `x-api-key` to a native WebSocket. The relay injects the key, validates Origin and Host, and forwards JSON frames in both directions.

## Production Flow

1. Open the upstream WebSocket with `voice-agent.zoom.v2` and server-side `x-api-key`.
2. Wait for `session.created`, send the `listen` / `think` / `speak` session update, and wait for `session.updated`.
3. Stream base64 PCM16 mono audio in `input_audio_buffer.append` frames.
4. Handle transcription, response, tool-call, and audio-delta events.
5. On speech start, interrupt queued playback and truncate the active item when appropriate.
6. Execute each function call once, send `function_call_output`, then request the next response.
7. Enforce backpressure and bounded playback buffering.

## Guardrails

- Match the session body to the subprotocol: V2 uses `listen` / `think` / `speak`; V1 uses `audio.input` / `audio.output` and returns `{type, payload}` events.
- Do not expose the API key to browser JavaScript.
- Do not send microphone frames before `session.updated`.
- Do not treat cancelled, failed, or incomplete responses as successful playback.
- A 404 on upgrade usually means the project is not provisioned for Voice Agent; verify Build Account or Universal Credit access and the `/v2/ws/aiservices/...` path.
