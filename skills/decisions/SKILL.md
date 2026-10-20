---
name: decisions
description: "Zoom AI Services Decisions for typed predicates, constrained classification, ordered scoring, probability-aware routing, and inline image comparison. Use when building or debugging Decisions requests, production x-api-key authentication, standard/pro model selection, Boolean choices, or score-label normalization."
triggers:
  - "zoom decisions"
  - "decisions api"
  - "decision probability"
  - "predicate classification"
  - "boolean choice"
  - "compare two images"
  - "zoom-decisions-standard"
  - "zoom-decisions-pro"
---

# Zoom AI Services Decisions

Use Decisions when an application needs a typed answer and probability distribution rather than open-ended generated text.

**Current API audit:** The production contract was live-tested on 2026-10-17 and rechecked against the AI Services OpenAPI catalog on 2026-10-20. The public catalog now includes `POST /aiservices/decisions/decide`; re-test when the specification changes.

## Routing Guardrail

- Use `predicate` for the probability that one explicit condition is true.
- Use `choice` for exactly one result from a caller-controlled closed set.
- Use `score` for a probability-weighted position across ordered levels.
- Use structured `input_image` content for visual classification or an explicit comparison such as “Is image A sharper than image B?”
- Do not use Decisions for open-ended writing, fact lookup, or unconstrained extraction.
- Do not treat the model as an application safety or authorization boundary.
- Route transcription to [Scribe](../scribe/SKILL.md), summaries to [Summarizer](../summarizer/SKILL.md), and translation to [Translator](../translator/SKILL.md).

## Production Contract

```http
POST https://api.zoom.us/v2/aiservices/decisions/decide
x-api-key: $ZOOM_AI_SERVICES_API_KEY
Content-Type: application/json
```

Required key scope: `aiservices:write:decide:admin`.

Retrieve the production `x-api-key` from a **Zoom Build Platform** project. Build Platform was previously known as the **Video SDK account**, and the project can be associated with a CPaaS account under a Zoom Meetings account, a standalone Build Platform account through `marketplace.zoom.us`, or a CPaaS account through [Platform Studio](https://platform.zoom.us/). The `zoom.ai` entry point may route users into the same project setup. Keep the key server-side and never expose it to browser code, logs, source control, or agent output.

Use a full endpoint override only when the operator explicitly targets another environment. Production accepts `x-api-key`, bearer with the API key, and bearer with a Zoom AI Services JWT; all three returned 200 on 2026-10-20. Do not silently send a production key to an internal host or an internal token to production.

Observed model inputs:

- `zoom-decisions-standard` — accepted alias and preferred default for ordinary workloads; the service may normalize the response model to `zoom-decisions`.
- `zoom-decisions-pro` — use when the operator requests the higher-capability model or evaluation shows it is needed.
- `zoom-decisions` — accepted service alias.

The response may report `zoom-decisions` after `zoom-decisions-standard` was requested. Treat that as alias normalization, not a failed model selection.

## Docs versus production

Rechecked 2026-10-20 against the docs and live requests. The docs are wrong or incomplete on these points:

- `name` is documented as optional. A question without `name` returns `400 INVALID_INPUT` with a generic payload message, so always send a unique name.
- `confidence` appears in the documented choice and score responses. Production responses contain no `confidence` field.
- Documented score probabilities include `label`. Production returns only `value` and `probability`.
- The docs say a Boolean choice returns a boolean. `choice` is boolean, but `probabilities[].value` is the string `"true"` or `"false"`.
- The docs describe JWT bearer auth only. `x-api-key` works in production and was verified, as does bearer with the API key.

The docs are accurate on question types, question independence, image opt-in, score as a weighted average, `reference_id` not being echoed, and the 400 error envelope.

## Core Workflow

1. Define a non-empty input and the smallest useful set of independent questions.
2. Give every question a trimmed, non-empty, unique `name`; do not depend on schemas that mark it optional.
3. Choose `predicate`, `choice`, or `score` and validate its type-specific fields before sending.
4. Set `enable_image_input: true` whenever any `input_image` part is present.
5. Send the request from a server-side wrapper with a timeout and bounded retries for transient responses only.
6. Match answers to questions by both order and `name`; allow `refusal` without assuming it is a safety control.
7. Normalize Boolean choice distributions and score labels against the original request before application logic consumes them.

## Response Semantics

- Predicate `probability` is the likelihood that the predicate is true.
- Choice returns one `choice` plus a probability for every allowed value.
- Score returns a probability-weighted average of zero-based level indices; it may fall between levels and is not a selected index. Probabilities are keyed by numeric index.
- No separate `confidence` field was observed. Never rename the winning probability to `confidence`.
- `reference_id` was accepted but not echoed in observed production responses.

Read [references/api-contract.md](references/api-contract.md) before implementing payload validation, image input, retries, or response normalization. Read [scenarios/high-level-scenarios.md](scenarios/high-level-scenarios.md) when choosing among escalation, routing, scoring, image comparison, multi-question, or media-to-decision patterns.

## Production Wrapper Requirements

- Default to the production endpoint and `x-api-key`; make non-production endpoint/auth overrides explicit.
- Validate locally so known `400 INVALID_INPUT` cases never consume a request.
- Apply a finite timeout.
- Retry only transient `429`, `503`, and `529` responses with bounded exponential backoff.
- Respect `Retry-After` and `metadata.retry_after_seconds` when supplied.
- Do not retry `400` or `401` responses automatically.
- Preserve the raw response for diagnostics while exposing a normalized view to callers.
- Log request IDs, status, latency, and retry count, but never input images or credentials by default.

## Chaining

- Cross-product use case → [Decisions Classification and Scoring](../general/use-cases/decisions-classification-scoring.md)
- One media file to transcript plus summary and classification → [Speech Summarization](../speech-summarization/SKILL.md) + `decisions`
- Media to transcript only, then classification → [Scribe](../scribe/SKILL.md) + `decisions`
- Live spoken conversation with typed decisions → [Voice Agent](../voice-agent/SKILL.md) + `decisions`
- Transcript recap plus typed escalation decision → [Summarizer](../summarizer/SKILL.md) + `decisions`
- Deterministic API wrapper and service inventory → `decisions` + [REST API](../rest-api/SKILL.md)
- Cross-product selection → [Zoom General](../general/SKILL.md)

## Sources and Tracking

- Docs: https://developers.zoom.us/docs/ai-services/decisions/fast-mode/
- Catalog: https://developers.zoom.us/api/zoap/ai-services/methods/
- AI Services quickstart: https://github.com/zoom/ai-services-quickstart/tree/wave-4
- Required question names: https://github.com/zoom/ai-services-quickstart/issues/4
- Type-specific validation: https://github.com/zoom/ai-services-quickstart/issues/5
- Boolean response inconsistency: https://github.com/zoom/ai-services-quickstart/issues/6
