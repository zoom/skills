# High-Level Scenarios

## Meeting Recap from Stored Transcript

Use fast mode when your app already has one transcript string and needs a quick recap:

```text
transcript text
  -> backend signs Build JWT
  -> POST /aiservices/summarizer/summarize
  -> persist result.text
  -> display recap in app
```

## Action Item Extraction

Use `task: "action_items"` when the user needs owner-grouped follow-ups. Store the original transcript and the rendered action items together so humans can audit the output.

## Full Summary Pipeline

Use `task: "full_summary"` to get recap, detailed summary, and action items in one rendered output. This is the best default for meeting or support-call review screens.

## Batch Transcript Archive Summaries

Use batch mode when transcripts live in S3 as `.vtt`, `.srt`, or `.txt` files:

```text
S3 transcript prefix
  -> POST /aiservices/summarizer/jobs
  -> poll job and file status or receive webhook
  -> read summary output files from S3
```

## Media-to-Text-to-Summary

When the input is media, chain:

```text
recording or uploaded media
  -> scribe
  -> transcript text or transcript files
  -> summarizer
  -> recap / summary / action items
```

## Localized Summary

Summarizer can return supported output languages directly. If the target language or workflow needs translation after summary generation, chain [../../translator/SKILL.md](../../translator/SKILL.md).
