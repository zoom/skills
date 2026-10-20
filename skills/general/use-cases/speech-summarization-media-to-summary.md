# Speech Summarization: Media to Transcript and Summary

Use this workflow when one uploaded audio or video file must produce both a transcript and a summary.

## Skill Chain

- Request shape, SSE events, limits, and failure handling: [Speech Summarization](../../speech-summarization/SKILL.md)
- Official Node.js sample: [Speech Summarization Node example](../../speech-summarization/examples/node.md)
- Typed routing after the summary: [Decisions](../../decisions/SKILL.md)
- Transcript-only workflows: [Scribe](../../scribe/SKILL.md)
- Existing transcript text: [Summarizer](../../summarizer/SKILL.md)

## Route by Input

| Input or outcome | Use |
|---|---|
| One audio/video file needs transcript plus summary | `speech-summarization` |
| Only a transcript is needed | `scribe` |
| Transcript text already exists | `summarizer` |
| Live meeting audio/video | `rtms` |
| Interactive microphone conversation | `voice-agent` |

## Production Flow

1. Keep the API key server-side and validate file type, size, and duration locally.
2. Send multipart `config` and `file` to `POST /v2/aiservices/speech-summarization/summarize`.
3. Parse SSE line-by-line; preserve multi-line data fields and UTF-8.
4. Print transcript text after `transcribe_complete`.
5. Treat summary frames as accumulated text and emit only the new suffix.
6. Require `done`; treat `error` or `response_state: "failed"` as failure.
7. Store request metadata and the final summary for audit.

## Guardrails

- Do not use the older playground payload's extra `model` or `summary_length` fields unless the current API contract requires them.
- Do not interpret `402 QUOTA_EXCEEDED` as a malformed request.
- Do not stream a browser upload directly from the client when the API key must remain secret; proxy through a server.
- Do not use this for live audio; use RTMS or Voice Agent instead.
