---
name: scribe
description: "Zoom AI Services Scribe for synchronous and batch transcription of uploaded or stored media. Use for Build-platform JWT auth, fast mode transcription, batch S3 jobs, webhook callbacks, and transcript-pipeline design."
triggers:
  - "scribe"
  - "ai services scribe"
  - "zoom scribe"
  - "transcribe audio file"
  - "transcribe video file"
  - "batch transcription"
  - "fast mode transcription"
  - "build platform jwt"
---

# Zoom AI Services Scribe

**Current API audit:** Official docs and the OpenAPI catalog were rechecked on 2026-10-20. Scribe Live is available at `GET /aiservices/scribe/live`; the verified WebSocket URL is `wss://api.zoom.us/v2/aiservices/scribe/live` with the `live-asr` subprotocol.

Implementation guidance for Zoom AI Services Scribe across:
- synchronous single-file transcription (`POST /aiservices/scribe/transcribe`)
- asynchronous batch jobs (`/aiservices/scribe/jobs*`)
- real-time microphone transcription over the Scribe Live WebSocket
- browser microphone pseudo-streaming via repeated short file uploads when Live is unavailable
- webhook-driven batch status updates
- Build-platform JWT generation and credential handling

Official docs:
- https://developers.zoom.us/docs/ai-services/
- https://developers.zoom.us/docs/ai-services/scribe/
- https://developers.zoom.us/docs/api/ai-services/
- https://developers.zoom.us/api/zoap/ai-services/methods/
- Quickstart sample: https://github.com/zoom/ai-services-quickstart/

## Routing Guardrail

- If the user needs **uploaded or stored media transcribed into text**, route here first.
- If the user needs **real-time microphone transcription**, use Scribe Live.
- If one uploaded media file needs **transcript plus summary**, route to [Speech Summarization](../speech-summarization/SKILL.md) instead of chaining Scribe and Summarizer.
- If the user needs **transcript text summarized**, route to [Summarizer](../summarizer/SKILL.md).
- If the user needs **plain text translated**, route to [../translator/SKILL.md](../translator/SKILL.md).
- If the user needs **live meeting media** with meeting metadata, participant identity, or meeting-scoped streams, route to [RTMS](../rtms/SKILL.md).
- If the user needs an **interactive spoken response**, route to [Voice Agent](../voice-agent/SKILL.md).
- If the user needs **Zoom REST API inventory** for AI Services paths, chain [../rest-api/SKILL.md](../rest-api/SKILL.md).
- If the user needs webhook signature patterns or generic HMAC receiver hardening, optionally chain [../webhooks/SKILL.md](../webhooks/SKILL.md).

## Quick Links

1. [concepts/auth-and-processing-modes.md](concepts/auth-and-processing-modes.md)
2. [scenarios/high-level-scenarios.md](scenarios/high-level-scenarios.md)
3. [examples/fast-mode-node.md](examples/fast-mode-node.md)
4. [examples/batch-webhook-pipeline.md](examples/batch-webhook-pipeline.md)
5. [references/api-reference.md](references/api-reference.md)
6. [references/environment-variables.md](references/environment-variables.md)
7. [references/samples-validation.md](references/samples-validation.md)
8. [references/versioning-and-drift.md](references/versioning-and-drift.md)
9. [troubleshooting/common-drift-and-breaks.md](troubleshooting/common-drift-and-breaks.md)
10. [RUNBOOK.md](RUNBOOK.md)

## Core Workflow

1. Get Build-platform credentials and generate an HS256 JWT.
2. Choose **fast mode** for one short file or **batch mode** for stored archives / large sets.
3. Submit the transcription request.
4. For batch jobs, poll job/file status or receive webhook notifications.
5. Persist and post-process transcript JSON.

## Hosted Fast-Mode Guardrail

- The formal fast-mode API limits are `100 MB` and `2 hours`, but hosted browser flows can still time out before the upstream response returns.
- Current deployed-sample observations:
  - ~17.2 MB MP4 completed in about `26s`
  - ~38.6 MB MP4 completed in about `26-37s`
  - ~59.2 MB MP4 completed in about `32-34s` on the backend
  - some ~59.2 MB browser requests still surfaced as frontend `504` while backend logs later showed `200`
- Treat frontend `504` plus backend `200` as a browser/edge timeout race, not an automatic transcription failure.
- For hosted UIs, prefer an async request/polling wrapper for fast mode instead of holding the browser open for the full upstream response.
- For larger or less predictable media, prefer batch mode even when the file is still within the formal fast-mode size limit.

## Live Scribe

- Endpoint: `wss://api.zoom.us/v2/aiservices/scribe/live`
- Subprotocol: `live-asr`
- Required key scope: `aiservices:write:live_transcribe:admin`
- Authenticate upstream with `Authorization: Bearer <Zoom AI Services JWT>` as the official docs require. A Zoom AI Services JWT, an API key sent as `x-api-key`, and an API key sent as bearer each completed a live handshake on 2026-10-20. A browser WebSocket cannot set headers, so use a server-side relay.
- After the relay connects, send `session.update` with settings under `config`:

  ```json
  {
    "type": "session.update",
    "config": { "language": "en-US", "interim_results": true },
    "audio": { "format": "pcm16" }
  }
  ```

- Stream signed little-endian 16 kHz mono PCM16 as binary frames of roughly 100 ms. Do not add a WAV header and do not base64-encode binary audio.
- `transcription.delta` events arrive only when `config.interim_results` is `true`. Each delta carries accumulated partial text for its `item_id`; replace the displayed partial instead of appending it.
- `transcription.completed` carries `item_id`, `transcript`, timing, and `speaker` when diarization is enabled.
- Close with `{"type":"session.close"}` and keep reading until pending transcript items drain and `session.closed` arrives.
- Session limits: 120 minutes maximum duration and a 30 second idle timeout with no audio events.

## Browser Microphone Fallback

- If Live is unavailable, pseudo-stream short chunks through the fast-mode file endpoint:
  1. capture microphone audio in short chunks
  2. upload each chunk through the async fast-mode wrapper
  3. poll for completion
  4. append chunk transcripts in sequence
- Recommended starting cadence:
  - chunk size: `5 seconds`
  - acceptable range: `5-10 seconds`
  - in-flight chunk requests: `2-3`
- This is a practical UI pattern for incremental transcript updates, not a substitute for `rtms`.
- Prefer Scribe Live for a true live-transcription product; keep pseudo-streaming as a fallback.
- Treat this as a fallback demo pattern, not the preferred production architecture.
- It adds repeated upload overhead, chunk-boundary drift, browser codec/container variability, and transcript stitching complexity.
- If the user asks for actual live stream ingestion, low-latency continuous media, or server-push media transport, route to [../rtms/SKILL.md](../rtms/SKILL.md) instead.

## Endpoint Surface

| Mode | Method | Path | Use |
|------|--------|------|-----|
| Fast | `POST` | `/aiservices/scribe/transcribe` | Synchronous transcription for one file |
| Live | `GET` | `/aiservices/scribe/live` | Real-time WebSocket transcription (`live-asr`) |
| Batch | `POST` | `/aiservices/scribe/jobs` | Submit asynchronous batch job |
| Batch | `GET` | `/aiservices/scribe/jobs` | List jobs |
| Batch | `GET` | `/aiservices/scribe/jobs/{jobId}` | Inspect job summary/state |
| Batch | `DELETE` | `/aiservices/scribe/jobs/{jobId}` | Cancel queued/processing job |
| Batch | `GET` | `/aiservices/scribe/jobs/{jobId}/files` | Inspect per-file results |
| Batch | `GET` | `/aiservices/scribe/jobs/{jobId}/files/{fileId}` | Inspect one per-file result |

## High-Level Scenarios

- On-demand clip transcription after a user uploads one recording.
- Batch transcription of stored S3 call archives.
- Webhook-driven ETL pipeline that writes transcripts to your database/search index.
- Re-transcription of Zoom-managed recordings after exporting them to your own storage.
- Offline compliance or QA workflows that need timestamps, channel separation, and speaker hints.

## Chaining

- Stored Zoom recordings -> [../rest-api/SKILL.md](../rest-api/SKILL.md) + `scribe`
- Transcribe then summarize -> `scribe` + [Summarizer](../summarizer/SKILL.md)
- One media file to transcript plus summary -> [Speech Summarization](../speech-summarization/SKILL.md)
- Transcribe then translate -> `scribe` + [../translator/SKILL.md](../translator/SKILL.md)
- Webhook verification hardening -> [../webhooks/SKILL.md](../webhooks/SKILL.md)
- Real-time live transcript/media -> [../rtms/SKILL.md](../rtms/SKILL.md)
- Interactive spoken response -> [Voice Agent](../voice-agent/SKILL.md)
- Cross-product routing -> [../general/SKILL.md](../general/SKILL.md)

## Operations

- [RUNBOOK.md](RUNBOOK.md) - 5-minute preflight and debugging checklist.
