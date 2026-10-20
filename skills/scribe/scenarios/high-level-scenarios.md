# High-Level Scenarios

## Scenario 1: On-Demand Upload Transcription

Use fast mode when a user uploads one file and expects a transcript immediately.

Flow:
1. Browser uploads file to your backend.
2. Backend generates Build JWT.
3. Backend calls `POST /aiservices/scribe/transcribe`.
4. Backend returns transcript JSON to the caller.

Common downstream uses:
- post-call summaries
- ticket enrichment
- searchable clip libraries
- internal review or handoff notes

## Scenario 2: Batch S3 Archive Transcription

Use batch mode when call archives or media libraries already live in S3.

Flow:
1. Build a batch request with input prefix and output prefix.
2. Submit `POST /aiservices/scribe/jobs`.
3. Track state by webhook or polling.
4. Read `/jobs/{jobId}/files` for per-file success/failure.
5. Ingest outputs into search, analytics, or storage.

Common downstream uses:
- compliance and audit logging
- searchable webinar or podcast archives
- bulk transcript backfills
- QA scoring inputs

## Scenario 3: Zoom Recording Export + Re-Transcription

Use when you must re-process Zoom-managed recordings with your own transcript settings.

Skill chain:
- `zoom-rest-api` to fetch/download recordings
- `scribe` to transcribe exported media

Typical reasons:
- you need your own retention/search pipeline
- you need different transcript settings than Zoom-managed defaults
- you want to enrich recordings with your own summarization or tagging flow

## Scenario 4: Compliance / QA Processing

Use batch mode when transcripts must be generated offline for audits, QA scoring, or archival search.

Prefer:
- `word_time_offsets=true` when reviewers need precise excerpts
- `channel_separation=true` for stereo call recordings
- webhook + queue ingestion instead of synchronous polling for large volumes

## Scenario 5: Customer Support Voice-to-Insights Pipeline

Use when support call recordings should feed operational analytics instead of stopping at raw transcript text.

Flow:
1. Ingest call recordings from storage or exported meeting assets.
2. Transcribe with `scribe`.
3. Store transcript plus speaker/timing metadata.
4. Run downstream sentiment, keyword, escalation, or QA logic in your own pipeline.

Guardrail:
- keep `scribe` focused on transcription
- do sentiment analysis, keyword detection, or scoring in downstream services after transcript generation

## Scenario 6: Live Microphone Transcription

Use Scribe Live when a web or native app should stream microphone audio and receive transcription events in real time.

Flow:
1. Browser captures 16 kHz mono PCM16 audio.
2. A server-side relay opens `wss://api.zoom.us/v2/aiservices/scribe/live` with `live-asr` and authenticates with a Zoom AI Services JWT bearer token, or an AI Services API key.
3. Relay sends `session.update` with `config.language`, optional `config.interim_results`, and `audio.format`, then forwards binary PCM16 frames.
4. Forward `transcription.delta` (only when interim results are enabled), `transcription.completed`, `session.closed`, and `error` events to the client.
5. On stop, send `session.close` and keep reading until `session.closed` arrives.

Guardrail:
- use RTMS instead when meeting metadata, participants, or meeting-scoped media are required
- use Voice Agent instead when the system must respond with speech
- replace accumulated `transcription.delta` text per `item_id` instead of appending it

## Scenario 7: Browser Microphone Incremental Transcript Fallback

Use pseudo-streaming only when Scribe Live is unavailable or the deployment cannot support a WebSocket relay.

Flow:
1. Browser captures microphone audio with `MediaRecorder`.
2. Browser flushes one chunk every `5 seconds`.
3. Backend accepts each chunk as a normal fast-mode upload through the async wrapper.
4. Frontend polls by request ID and appends transcript chunks in order.

Guardrail:
- this is pseudo-streaming over repeated file uploads
- this is best kept as a lightweight demo or constrained fallback
- prefer Scribe Live for a true live-transcription product
- if the requirement is truly live media stream ingestion or lower-latency continuous audio, route to `rtms`
