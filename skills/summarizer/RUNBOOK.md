# Summarizer 5-Minute Preflight Runbook

Use this before deep debugging.

## 1. Confirm the Right Product

- Transcript or dialogue text to recap/summarize -> stay on `summarizer`.
- Audio/video media to text -> use `scribe` first.
- Live meeting media -> use `rtms` first.
- Plain text translation -> use `translator`.

## 2. Confirm Credentials

- Build-platform API key and API secret are present.
- JWT uses `HS256`.
- `iss` is the API key.
- `iat` and `exp` are epoch seconds.
- Expiry is one hour or less for better security.
- Secret stays server-side.

## 3. Confirm Mode Selection

- **Fast mode** for one inline transcript string.
- **Batch mode** for stored transcript files, archives, or offline pipelines.

Fast mode limit:
- input text: `96 KB` maximum

Batch mode limits:
- prefix job: `10,000` files maximum
- manifest job: `1,000` files maximum
- file size: `96 KB` maximum
- characters per job: `100,000,000` maximum
- input formats: `.vtt`, `.srt`, `.txt`

## 4. Confirm Request Shape

- Fast mode requires `input.text` and `config`.
- Batch mode requires storage input/output details and `config`.
- `config.summary_type` is currently `conversation`.
- `config.task` is one of `recap`, `summary`, `action_items`, `full_summary`.
- `config.language` is a supported BCP-47 locale code.
- Do not add `output_format`; current docs and quickstart source removed it for Summarizer.

## 5. Confirm Storage and Webhooks

- S3 URI exists and permissions are valid.
- Temporary AWS credentials have not expired.
- Webhook URL is public HTTPS if notifications are expected.
- Webhook verification uses raw request body, `x-zm-signature`, and `x-zm-request-timestamp`.

## 6. Quick Probes

- Generate a JWT locally.
- Submit a tiny fast-mode transcript to `POST /aiservices/summarizer/summarize`.
- Submit a one-file batch job from S3.
- Poll `/jobs/{jobId}`.
- Poll `/jobs/{jobId}/files`.

## 7. Fast Decision Tree

- `401` -> wrong Build credential pair or expired JWT.
- `400` on fast mode -> invalid task/language or text over `96 KB`.
- Batch job queued forever -> storage auth, URI, or service backlog.
- Webhook missing -> URL not public HTTPS or notification object omitted.
- Webhook signature mismatch -> raw body changed before verification.

## 8. Source Checkpoints

- https://developers.zoom.us/docs/ai-services/summarizer/
- https://developers.zoom.us/docs/ai-services/summarizer/fast-mode/
- https://developers.zoom.us/docs/ai-services/summarizer/batch-mode/
- https://developers.zoom.us/api-hub/ai-services/methods/endpoints.json
- https://github.com/zoom/ai-services-quickstart/
