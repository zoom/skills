# Samples Validation

Validated against:
- https://github.com/zoom/ai-services-quickstart/
- official Translator docs under `docs/ai-services/translator/`
- AI Services OpenAPI inventory at `api-hub/ai-services/methods/endpoints.json`
- Zoom blog: `build-multilingual-experiences-with-zoom-translator-api`

## Sample Age

The official quickstart repository had latest commit `a5679718ce407d63a0d56b34d4627ba9ba509193` on `2026-05-18`, so it is current relative to the 3-year sample-age rule.

## What the Sample Confirms

- A Node/Express backend plus React playground is a valid reference architecture.
- JWT generation and Zoom API calls belong server-side.
- Fast mode routes call `/aiservices/translator/translate`.
- Batch helper routes can share the same job polling, file polling, cancellation, storage-auth, and webhook patterns as Scribe and Summarizer.
- Environment variable names are `ZOOM_API_KEY`, `ZOOM_API_SECRET`, AWS credential keys, and `WEBHOOK_SECRET`.

## Caveats

- The README setup command still says to clone `zoom/scribe-quickstart.git`; the current repo name is `zoom/ai-services-quickstart.git`.
- Current docs list `zh-TW` as a supported Translator locale. The sampled router's hardcoded language list did not include `zh-TW` when inspected, so do not let the sample override the official docs.
- The sample uses Node `>=24`; do not impose that on every production implementation unless the chosen app stack requires it.
- Treat the sample as implementation guidance, not the source of truth for endpoint inventory. Use the docs and API Hub for request shapes.
