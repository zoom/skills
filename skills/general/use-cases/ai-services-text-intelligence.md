# AI Services Text Intelligence

Use Zoom AI Services when the workflow turns media or text into transcripts, summaries, action items, or translations through Build-platform JWT-authenticated APIs.

## Skill Chain

- Media to transcript: [../../scribe/SKILL.md](../../scribe/SKILL.md)
- Transcript to recap, summary, or action items: [../../summarizer/SKILL.md](../../summarizer/SKILL.md)
- Text localization: [../../translator/SKILL.md](../../translator/SKILL.md)
- Endpoint inventory: [../../rest-api/SKILL.md](../../rest-api/SKILL.md)
- Batch webhook hardening: [../../webhooks/SKILL.md](../../webhooks/SKILL.md)

## Route by Input

| Input | Primary skill |
|-------|---------------|
| Audio/video file | `scribe` |
| Transcript or conversation text | `summarizer` |
| Plain text to localize | `translator` |
| Live meeting media | `rtms`, then post-process with AI Services if needed |

## Common Chains

1. Recording summary
   - download/export recording
   - transcribe with `scribe`
   - summarize with `summarizer`

2. Localized action items
   - summarize transcript with `task: "action_items"`
   - translate rendered action items with `translator`

3. Batch archive processing
   - submit S3 media files to Scribe batch
   - submit transcript files to Summarizer batch
   - optionally submit text files to Translator batch
   - monitor each job with polling or webhooks

## Credential Model

All three AI Services products use Build-platform API key and API secret to generate HS256 JWTs. Do not use standard Zoom OAuth tokens for these endpoints unless Zoom changes the product docs.

## Batch Webhook Pattern

The batch products share the same webhook verification shape:

```text
message = "v0:{x-zm-request-timestamp}:{raw_body}"
expected = "sha256=" + HMAC_SHA256(message, notifications.secret)
compare expected to x-zm-signature with a timing-safe comparison
```

Verify the raw request body before parsing JSON.

## Design Notes

- Keep Zoom Build credentials server-side.
- Use fast mode for bounded single inputs.
- Use batch mode when inputs already live in cloud storage or may exceed fast-mode limits.
- Treat official docs and API Hub as authoritative. Use sample apps for architecture and request patterns after checking age and drift.
