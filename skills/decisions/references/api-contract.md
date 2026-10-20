# Decisions API contract and observed workarounds

This reference records production behavior live-tested on 2026-10-17 and rechecked against the public AI Services OpenAPI catalog on 2026-10-20. The endpoint is now part of the published catalog.

## Minimal request

```json
{
  "model": "zoom-decisions-standard",
  "input": "All customers are unable to sign in.",
  "questions": [
    {
      "type": "predicate",
      "name": "needs_escalation",
      "instructions": "Does this issue require immediate escalation?"
    }
  ]
}
```

## Validation matrix

| Field | Observed requirement |
|---|---|
| `input` | Non-empty string, or non-empty user-message array with non-empty content parts |
| `questions` | At least one question |
| `name` | Required, trimmed, non-empty, and unique across questions |
| `instructions` | Non-empty string |
| Choice values | At least two unique non-empty strings or native Booleans |
| Score levels | At least two levels with non-empty labels |
| Images | `enable_image_input: true` is required |

Whitespace-only question names were accepted in one test but are a validation gap, not a supported unnamed-question workaround. Boolean `true` and string `"true"` collide after canonical string conversion; reject that pair locally.

## Structured and image input

```json
{
  "input": [
    {
      "role": "user",
      "content": [
        {"type": "input_text", "text": "Image A follows."},
        {"type": "input_image", "image_url": "data:image/png;base64,...", "detail": "high"},
        {"type": "input_text", "text": "Image B follows."},
        {"type": "input_image", "image_url": "data:image/png;base64,...", "detail": "high"}
      ]
    }
  ],
  "enable_image_input": true,
  "questions": [
    {
      "type": "predicate",
      "name": "same_subject",
      "instructions": "Do image A and image B show the same primary subject?"
    }
  ]
}
```

Observed working inputs include inline JPEG and PNG data URLs, image-only input, text plus image, two images in one message, and two images across two user messages. Observed `detail` values are `auto`, `low`, `high`, and `original`.

Validate local paths, supported media types, and input size before converting a file to a data URL. Do not fetch arbitrary user-supplied URLs from a privileged server without an SSRF policy.

## Boolean normalization

A request can use native Boolean values:

```json
"choices": [{"value": true}, {"value": false}]
```

An observed response mixed types:

```json
{
  "choice": true,
  "probabilities": [
    {"value": "true", "probability": 0.99},
    {"value": "false", "probability": 0.01}
  ]
}
```

Build a map from canonical `String(requestValue)` to the original value, then restore each returned probability value. Reject canonical collisions in the request first.

## Score labels

Score responses use numeric indexes rather than request labels:

```json
{
  "score": 1.94,
  "probabilities": [
    {"value": 0, "probability": 0.02},
    {"value": 1, "probability": 0.02},
    {"value": 2, "probability": 0.96}
  ]
}
```

Map each numeric `value` to `questions[].levels[value].label`. Keep the weighted `score` numeric; do not round it before business logic unless the product explicitly needs the nearest level.

## Batches and refusals

Mixed predicate, choice, and score questions work in one request and observed answers preserve order. Validate that answer names and types match the corresponding questions. Accept `type: "refusal"` as a possible per-question result, but enforce application safety policy independently because protected-class probes did not reliably produce refusals.

## Error and retry handling

Typical service errors contain `code`, `reason`, `message`, and `metadata`. Retry only transient `429`, `503`, and `529` failures, cap attempts, and honor server retry timing. Treat invalid JSON or an answer count/name/type mismatch as an upstream contract error and retain the request ID for diagnosis.
