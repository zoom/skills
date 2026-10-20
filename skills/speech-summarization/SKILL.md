---
name: speech-summarization
description: "Zoom AI Services Speech Summarization for one-shot audio or video uploads that return a transcript plus streamed summary over SSE. Use for media-to-summary in one request; not for transcript-only Scribe, transcript-text Summarizer, live meeting media, or interactive Voice Agent."
triggers:
  - "speech summarization"
  - "speech summary"
  - "summarize audio file"
  - "summarize video file"
  - "audio transcript and summary"
  - "transcribe and summarize"
  - "speech-summarization api"
---

# Zoom AI Services Speech Summarization

Use Speech Summarization when one uploaded audio or video file must produce both a transcript and a summary in a single streaming request.

**Current API audit:** The production endpoint and multipart request were verified on 2026-10-20. Live calls were accepted but returned `402 QUOTA_EXCEEDED` before streaming, so the SSE event sequence below comes from the API reference and official sample rather than an observed end-to-end run.

## Routing Guardrail

- One media file needs **transcript plus summary** → use this skill.
- Only a transcript is needed → use [Scribe](../scribe/SKILL.md).
- Transcript text already exists and only a recap, summary, or action items are needed → use [Summarizer](../summarizer/SKILL.md).
- Live meeting audio/video → use [RTMS](../rtms/SKILL.md).
- Interactive microphone conversation with spoken responses → use [Voice Agent](../voice-agent/SKILL.md).
- Typed classification, routing, scoring, or image decisions → use [Decisions](../decisions/SKILL.md).

## Production Contract

```http
POST https://api.zoom.us/v2/aiservices/speech-summarization/summarize
x-api-key: $ZOOM_AI_SERVICES_API_KEY
Accept: text/event-stream
Content-Type: multipart/form-data
```

Authentication also accepts `Authorization: Bearer $ZOOM_AI_SERVICES_API_KEY`. Keep the key server-side. The required key scope is `aiservices:write:transcribe-summarize:admin`.

The official sample reads the same credential from `ZOOM_API_KEY`. Use one name in your deployment; `ZOOM_AI_SERVICES_API_KEY` is this skill's preferred name because it distinguishes the API key from Build-platform JWT pairs.

Multipart fields:

- `config`: JSON string with `transcribe` and `summary` settings.
- `file`: WAV, MP3, M4A, FLAC, OGG, WebM, MP4, or MOV.

Formal limits are 50 MB and 2 hours. Use 8 kHz or higher audio; 16 kHz or higher is recommended. Mono or stereo is accepted.

## Workflow

1. Validate the media type, size, and duration locally before upload.
2. Build the multipart request server-side; never expose the API key in browser code.
3. Parse the SSE stream line-by-line, preserving UTF-8 and multi-line `data:` fields.
4. Handle `transcribe_complete` before consuming summary frames.
5. Treat each summary frame as accumulated text; emit only the new suffix to a UI.
6. Finish only after `done`. Treat `error` or `response_state: "failed"` as a failed request.

## Response Contract

The stream uses these events:

- `transcribe_started`
- `transcribe_complete`
- `summary_delta`
- `done`
- `error`

`transcribe_complete.transcript` may be a JSON string or an object. Summary text may appear under `summary_result`, `summary_result.summarization.overall_summary`, nested `metadata`, `summary`, or `text`. See [references/api-contract.md](references/api-contract.md) for exact parsing fallbacks.

## Failure Handling

- `401`: invalid key or missing `aiservices:write:transcribe-summarize:admin` scope.
- `402 QUOTA_EXCEEDED`: observed when the account's AI Services credits are exhausted. The catalog does not list `402`; treat it as an entitlement signal, not a schema error. Two different API keys returned it on 2026-10-20, Scribe fast mode returned the same code with `Free-trial credit exhausted`, and there is no `Retry-After`, so neither rotating keys nor retrying will clear it.
- `405`: wrong HTTP method; this operation is `POST` only.
- `413`: file exceeds 50 MB.
- `415`: unsupported media type.
- `429`: rate limit; honor retry timing.
- `502`: upstream transcription or summary failure.
- `503`: summary API is not configured.

Do not treat `402` as a malformed request. The request shape can be correct while the account lacks credits. Check the plan and credit balance in [Platform Studio](https://platform.zoom.us/); a new project key under the same account will not clear the 402. Decisions and Voice Agent turns still succeeded on the same key, so a 402 on metered transcription or summarization does not make the key unusable everywhere.

## Chaining

- One-shot media to typed routing → `speech-summarization` + [Decisions](../decisions/SKILL.md)
- One-shot media to localized output → `speech-summarization` + [Translator](../translator/SKILL.md)
- Transcript-only workflow → [Scribe](../scribe/SKILL.md)
- Existing transcript workflow → [Summarizer](../summarizer/SKILL.md)
- Live meeting media → [RTMS](../rtms/SKILL.md)
- Real-time spoken conversation → [Voice Agent](../voice-agent/SKILL.md)

## References

- [API contract and SSE parsing](references/api-contract.md)
- [Official Node.js sample](examples/node.md)
