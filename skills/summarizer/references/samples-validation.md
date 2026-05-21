# Samples Validation

Validated against:
- https://github.com/zoom/ai-services-quickstart/
- official Summarizer docs under `docs/ai-services/summarizer/`
- AI Services OpenAPI inventory at `api-hub/ai-services/methods/endpoints.json`
- Zoom blog: `extract-intelligence-from-transcripts-with-summarizer`

## Sample Age

The official quickstart repository had latest commit `a5679718ce407d63a0d56b34d4627ba9ba509193` on `2026-05-18`, so it is current relative to the 3-year sample-age rule.

## What the Sample Confirms

- A Node/Express backend plus React playground is a valid reference architecture.
- JWT generation and Zoom API calls belong server-side.
- Fast mode routes call `/aiservices/summarizer/summarize`.
- Batch helper routes can share the same job polling, file polling, cancellation, storage-auth, and webhook patterns as Scribe and Translator.
- Environment variable names are `ZOOM_API_KEY`, `ZOOM_API_SECRET`, AWS credential keys, and `WEBHOOK_SECRET`.

## Caveats

- The README setup command still says to clone `zoom/scribe-quickstart.git`; the current repo name is `zoom/ai-services-quickstart.git`.
- The quickstart `docs/api-snippets.md` still shows `output_format` for Summarizer. Current official docs and current source router omit that field, and the latest commit message says output format was removed from Summarizer. Do not copy `output_format` into new Summarizer guidance.
- The sample uses Node `>=24`; do not impose that on every production implementation unless the chosen app stack requires it.
- Treat the sample as implementation guidance, not the source of truth for endpoint inventory. Use the docs and API Hub for request shapes.
