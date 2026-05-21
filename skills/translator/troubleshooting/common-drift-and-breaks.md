# Common Drift and Breaks

## Unsupported Language Pair

Translator currently supports bidirectional translation with English. Make sure either `source_language` or the single target locale is `en-US`.

## Multiple Target Languages

Fast mode supports one target language per request, and batch mode supports one target language per job. Split multi-language work into separate requests/jobs.

## Short Locale Codes

Use BCP-47 locale codes such as `en-US`, `es-ES`, or `zh-CN`. Do not send `en`, `es`, or lowercase-only shortcuts unless the docs explicitly allow them for the field.

## Input Too Large

Fast mode accepts up to `4,000` characters. Batch files are limited to `16 KB` and `4,000` translatable characters per file.

## Wrong Input Type

Translator accepts plain text or text files, not raw audio/video. Use [../../scribe/SKILL.md](../../scribe/SKILL.md) first for media files.

## Batch Job Queued or Failed

Check:
- S3 URI and region
- temporary AWS credential expiry
- input file size
- one target language only
- English-pivot language constraint
- `filters.include_globs` and `filters.exclude_globs`

## Missing Webhooks

Check:
- webhook URL is public HTTPS
- `notifications.webhook_url` and `notifications.secret` were included at job creation
- receiver verifies raw body with `x-zm-signature`
- timestamp replay window is enforced
