# Decisions Classification and Scoring

Use Zoom AI Services Decisions when a workflow needs a typed result with probabilities rather than open-ended generated text.

## Skill Chain

- Request design, model choice, response semantics, and workarounds: [Decisions](../../decisions/SKILL.md)
- Detailed payload and normalization contract: [Decisions API contract](../../decisions/references/api-contract.md)
- Server-side REST wrapper patterns: [REST API](../../rest-api/SKILL.md)
- Media-to-decision pipelines: [Scribe](../../scribe/SKILL.md), then Decisions
- Transcript recap plus typed routing: [Summarizer](../../summarizer/SKILL.md), then Decisions

## Route by Outcome

| Required outcome | Question type | Example |
|---|---|---|
| Probability that one condition is true | `predicate` | Does this incident require immediate escalation? |
| Exactly one result from a fixed set | `choice` | Route to billing, support, sales, or security |
| Position across ordered levels | `score` | Rate severity from low through critical |
| Visual yes/no comparison | `predicate` with two `input_image` parts | Is image A sharper than image B? |

Use one request with multiple named questions when the same input needs independent predicate, choice, and score answers. Match returned answers by order and name.

## Production Flow

1. Retrieve an `x-api-key` from a Zoom Build Platform project associated with a CPaaS account under a Zoom Meetings account, a standalone Build Platform account through `marketplace.zoom.us`, or a CPaaS account through `aiservice.zoom.us`.
2. Store the key server-side as `ZOOM_AI_SERVICES_API_KEY`.
3. Validate non-empty input and type-specific question fields locally.
4. Call `POST https://api.zoom.us/v2/aiservices/decisions/decide` with `zoom-decisions-standard` unless evaluation justifies `zoom-decisions-pro`.
5. Normalize Boolean probability values and score labels against the original request.
6. Apply application-owned thresholds, review policy, and safety controls.

## Common Chains

1. Support ticket routing
   - use a choice question for the owning queue
   - use a predicate for immediate escalation
   - use a score for ordered severity

2. Voice-of-customer triage
   - transcribe media with Scribe
   - classify intent with Decisions
   - send low-margin outcomes to human review

3. Image quality gate
   - send two labeled inline images with `enable_image_input: true`
   - ask one explicit comparative predicate
   - retain probability and request ID for audit

## Guardrails

- Do not call the winning probability `confidence`; no separate confidence field was observed.
- Do not silently expand the caller's allowed choices.
- Do not round a weighted score before business logic unless the product explicitly needs the nearest level.
- Do not rely on refusal behavior as the safety or authorization boundary.
- Do not expose an API key or inline image payload in browser code or routine logs.

For concrete scenario selection, read [Decisions high-level scenarios](../../decisions/scenarios/high-level-scenarios.md).
