# Translator 5-Minute Preflight Runbook

Use this before deep debugging.

## 1. Confirm the Right Product

- Plain text translation -> stay on `translator`.
- Audio/video to translated text -> use `scribe` first.
- Transcript summary before translation -> use `summarizer` before `translator`.
- Live meeting media -> use `rtms` first.

## 2. Confirm Credentials

- Build-platform API key and API secret are present.
- JWT uses `HS256`.
- `iss` is the API key.
- `iat` and `exp` are epoch seconds.
- Expiry is one hour or less for better security.
- Secret stays server-side.

## 3. Confirm Mode Selection

- **Fast mode** for one text string.
- **Batch mode** for stored text files or offline translation pipelines.

Fast mode limit:
- input text: `4,000` characters maximum

Batch mode limits:
- file size: `16 KB` maximum
- characters per file: `4,000`
- manifest job: `1,000` files maximum
- translatable characters per job: `100,000,000`
- translatable characters per day: `1,000,000,000`

## 4. Confirm Request Shape

- Fast mode requires `text` and `config`.
- Batch mode requires storage input/output details and `config`.
- `config.target_languages` must contain exactly one target locale.
- Either source or target must be `en-US`.
- Do not use short language codes such as `en` or `es`.

## 5. Confirm Storage and Webhooks

- S3 URI exists and permissions are valid.
- Temporary AWS credentials have not expired.
- Webhook URL is public HTTPS if notifications are expected.
- Webhook verification uses raw request body, `x-zm-signature`, and `x-zm-request-timestamp`.

## 6. Quick Probes

- Generate a JWT locally.
- Submit a tiny fast-mode request to `POST /aiservices/translator/translate`.
- Submit a one-file batch job from S3.
- Poll `/jobs/{jobId}`.
- Poll `/jobs/{jobId}/files`.

## 7. Fast Decision Tree

- `401` -> wrong Build credential pair or expired JWT.
- `400` on fast mode -> unsupported language pair, multiple targets, or text over `4,000` characters.
- Batch job queued forever -> storage auth, URI, or service backlog.
- File rejected -> text too large or unsupported file layout.
- Webhook signature mismatch -> raw body changed before verification.

## 8. Source Checkpoints

- https://developers.zoom.us/docs/ai-services/translator/
- https://developers.zoom.us/docs/ai-services/translator/fast-mode/
- https://developers.zoom.us/docs/ai-services/translator/batch-mode/
- https://developers.zoom.us/api-hub/ai-services/methods/endpoints.json
- https://github.com/zoom/ai-services-quickstart/
