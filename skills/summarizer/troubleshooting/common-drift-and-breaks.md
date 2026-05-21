# Common Drift and Breaks

## `output_format` Rejected

Current Summarizer docs and quickstart source do not use `output_format`. Remove it from Summarizer requests even if an older snippet includes it.

## Input Too Large

Fast mode accepts up to `96 KB` of text. Split or store transcript files and use batch mode for larger workloads.

## Wrong Input Type

Summarizer accepts transcript/dialogue text, not raw audio/video. Use [../../scribe/SKILL.md](../../scribe/SKILL.md) first for media files.

## Language Code Mismatch

Use BCP-47 locale codes such as `en-US`, `zh-CN`, or `ja-JP`. Avoid short codes such as `en`.

## Batch Job Queued or Failed

Check:
- S3 URI and region
- temporary AWS credential expiry
- input file size
- supported extension: `.vtt`, `.srt`, `.txt`
- `filters.include_globs` and `filters.exclude_globs`

## Missing Webhooks

Check:
- webhook URL is public HTTPS
- `notifications.webhook_url` and `notifications.secret` were included at job creation
- receiver verifies raw body with `x-zm-signature`
- timestamp replay window is enforced
