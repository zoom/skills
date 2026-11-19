---
name: clinical-notes
description: "Zoom AI Services Clinical Notes for transcription plus template-based clinical notes. Use for SOAP, General Medicine, or Behavioral Health notes from recordings, synchronous generation, async URL jobs, and structured section output."
triggers:
  - "clinical notes"
  - "soap note"
  - "medical note"
  - "behavioral health note"
  - "clinical note api"
  - "transcribe consultation"
---

# Zoom AI Services Clinical Notes

Source audit: 2026-11-19 docs snapshot and `openapi-ai-services.json`.

Clinical Notes transcribes one recording and generates a clinical note. The synchronous endpoint returns transcript and note together; the asynchronous endpoint accepts a recording URL and is polled to completion.

| Mode | Endpoint | Input |
|---|---|---|
| Synchronous | `POST /aiservices/clinical_notes/generate` | Multipart upload or JSON recording URL |
| Asynchronous | `POST /aiservices/clinical_notes/job` | JSON recording URL only |
| Status | `GET /aiservices/clinical_notes/job/{jobId}` | UUID `jobId` |

Required granular scope from OpenAPI: `aiservices:write:clinical_notes:admin`. Synchronous rate-limit label: `LIGHT`.

## Templates

| Template | Structure |
|---|---|
| `SOAP` | Subjective, Objective, Assessment, Plan |
| `GENERAL_MEDICINE` | History, examination, assessment, plan, and related sections |
| `GENERAL_BEHAVIORAL_HEALTH` | Behavioral-health encounter, including Mental Status Exam |

Names are case-sensitive. Omitted, empty, or whitespace-only `template` defaults to `GENERAL_MEDICINE`. An unknown template returns HTTP `400`.

The OpenAPI also documents `GENERAL_MEDICAL` as an alias for `GENERAL_MEDICINE`; the how-to docs list only the three canonical names, so prefer a canonical value.

## Synchronous Generation

Provide exactly one recording, either as a multipart `file` part or as a downloadable HTTPS/S3 URI in JSON. Do not send both.

```bash
curl -X POST "https://api.zoom.us/v2/aiservices/clinical_notes/generate" \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@consultation.wav;type=audio/wav" \
  -F "template=SOAP"
```

```bash
curl -X POST "https://api.zoom.us/v2/aiservices/clinical_notes/generate" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"file":"https://example.com/consultation.wav","template":"SOAP"}'
```

Limits: `.wav`, `.mp3`, `.m4a`, `.mp4`, and `.webm`; at most 2 hours; 300 MB for uploads and 2 GB for URLs. The URI should have a supported extension; malformed or unsupported extensions can return `400 UNSUPPORTED_MEDIA`.

A successful response contains:

- `request_id`
- `transcript.text_display`, optional `duration_sec`, and ordered `segments` with time, channel, speaker, text, words, and optional language
- `clinical_notes.full_note`, Markdown for the complete note
- `clinical_notes.sections`, an ordered array of `title` and `content`

Display segments when present; fall back to `text_display` when segments are empty. A missing `duration_sec` does not mean zero duration.

The OpenAPI adds optional `metadata` with `language`, `dob`, `gender`, `pronoun`, and `user_time_zone`, plus up to 64 ordered `sections` overrides. A nonempty override must have an effective heading via `title` or `display_name`, may include `description`, `instructions`, and `length` (`concise` or `detailed`), and must have unique normalized headings. The how-to page does not document these extensions; verify them in a lower environment before relying on them.

## Asynchronous Jobs

Submit JSON with one accessible HTTPS or S3 recording URI. The job endpoint does not accept uploads, `file_url`, or `zoom_file_id`; use synchronous generation for an upload. Keep the recording available at the URL until the job finishes.

```bash
curl -X POST "https://api.zoom.us/v2/aiservices/clinical_notes/job" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"file":"https://example.com/consultation.wav","template":"SOAP"}'
```

Submission returns HTTP `200` with `job_id`, `state` (initially `QUEUED`), and `submitted_at`. Save the UUID `job_id`.

Poll:

```bash
curl "https://api.zoom.us/v2/aiservices/clinical_notes/job/$JOB_ID" \
  -H "Authorization: Bearer $TOKEN"
```

| State | Meaning | Final |
|---|---|---|
| `QUEUED` | Waiting to be processed | No |
| `PROCESSING` | Being processed | No |
| `SUCCEEDED` | Completed; response includes `transcript` and `clinical_notes` | Yes |
| `FAILED` | Transcription or note generation failed | Yes |

The status endpoint returns HTTP `200` for a known job. A job can be HTTP `200` while `state` is `FAILED`; branch on `state`, not the HTTP status. `error` appears on a failed job only when failure details are available. Other states do not include `error`.

## Errors

| Status | Meaning |
|---|---|
| `400 VALIDATION_MISSING_FIELD` | JSON request lacks `file`. |
| `400 INVALID_INPUT` | Multipart lacks `file`, template is unknown, recording is unreachable, or a setting is invalid. |
| `400 UNSUPPORTED_MEDIA` | Recording URI has no supported media extension. |
| `401` | Missing/invalid credentials; the body can use the platform error shape and lack `reason`. |
| `429` | Rate limited; honor `Retry-After`. |
| `500` / `503` / `504` | Processing/service/gateway failure; retry according to your policy. |

If either transcription or note generation fails, the entire synchronous request fails.

## Routing

- Transcript only, without a clinical note: [Scribe](../scribe/SKILL.md).
- General summary or action items: [Summarizer](../summarizer/SKILL.md).
- One-shot media transcript plus nonclinical summary: [Speech Summarization](../speech-summarization/SKILL.md).

## Sources

- Snapshot 2026-11-19: `developers.zoom.us/docs/ai-services/clinical-notes.md`
- Snapshot 2026-11-19: `developers.zoom.us/docs/ai-services/clinical-notes/generate.md`
- Snapshot 2026-11-19: `developers.zoom.us/docs/ai-services/clinical-notes/jobs.md`
- OpenAPI snapshot 2026-11-19: `developers.zoom.us/docs/api/ai-services/openapi-ai-services.json` (sync endpoint only)
