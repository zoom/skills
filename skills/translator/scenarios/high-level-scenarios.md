# High-Level Scenarios

## Chat Message Translation

Use fast mode for short plain text:

```text
message text
  -> backend signs Build JWT
  -> POST /aiservices/translator/translate
  -> read result.translations[targetLocale]
  -> display translated text
```

## UI String or Workflow Text Translation

Use fast mode when translating one short app string, action label, note, or workflow instruction. Keep the source and target locale with the stored translation so downstream systems can audit language choices.

## Batch Text Archive Translation

Use batch mode when text files live in S3:

```text
S3 text prefix
  -> POST /aiservices/translator/jobs
  -> poll job and file status or receive webhook
  -> read translated output JSON files from S3
```

## Transcript Translation

If the input is media, first transcribe it with [../../scribe/SKILL.md](../../scribe/SKILL.md), then send the transcript text or transcript files to Translator.

## Summary Localization

Generate summaries with [../../summarizer/SKILL.md](../../summarizer/SKILL.md), then translate the rendered summary if the desired language workflow is not already covered by Summarizer output language support.

## English-Pivot Guardrail

Translator currently supports bidirectional translation with English. For non-English to non-English workflows, design an explicit English-pivot flow only if product requirements accept the extra step and quality tradeoff.
