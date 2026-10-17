# High-Level Scenarios

## Scenario 1: Probability-Aware Escalation

Use a predicate when the application needs the likelihood that a clearly stated condition is true.

Flow:
1. Send the incident, ticket, or account state as input.
2. Ask one named predicate such as `needs_escalation`.
3. Compare `probability` with an application-owned threshold.
4. Route uncertain or high-impact cases to human review.

Guardrail: probability is not authorization and is not a separately returned confidence value.

## Scenario 2: Closed-Set Routing

Use a choice question when every allowed destination is known in advance.

Examples:
- billing, sales, support, or security queue
- approve, reject, or manual review
- one action from a game or workflow state machine

Require at least two unique non-empty values. Preserve native Boolean values when they are meaningful, and normalize stringified Boolean probability values against the request.

## Scenario 3: Ordered Severity or Quality Score

Use score when outcomes form an ordered scale such as low, medium, high, and critical.

Flow:
1. Define levels from lowest to highest.
2. Send one named score question.
3. Interpret `score` as a probability-weighted zero-based level index.
4. Map numeric distribution indexes back to request labels.

Do not assume the response will echo level labels.

## Scenario 4: Compare Two Images

Use structured input when a decision depends on visual evidence.

Flow:
1. Label image A and image B with adjacent `input_text` parts.
2. Send each image as an inline `input_image` data URL.
3. Set `enable_image_input: true`.
4. Ask an explicit predicate such as “Is image A sharper than image B?”

Validate file type and size locally. Do not fetch arbitrary URLs from a privileged backend without an SSRF policy.

## Scenario 5: Multi-Question Triage

Use one request when the same input needs independent outputs.

Example batch:
- predicate: should this be escalated?
- choice: which team owns it?
- score: what is the ordered severity?

Give every question a unique non-empty name. Verify answer count, order, name, and type before using any result.

## Scenario 6: Media-to-Decision Pipeline

Use this chain when the original evidence is audio or video:

1. Transcribe with [Scribe](../../scribe/SKILL.md).
2. Optionally summarize long conversational input with [Summarizer](../../summarizer/SKILL.md).
3. Send the transcript or summary to Decisions for a typed outcome.
4. Store raw and normalized results with request IDs for operational review.

Avoid repeatedly calling Decisions on the same successful input; calls consume AI Services usage.
