# Scribe Text Polishing

Text Polishing converts a Scribe transcript into clearer written text while preserving names, facts, terminology, and intended meaning. It can remove filler words, false starts, and repetition; improve grammar, punctuation, capitalization, and readability; condense verbose speech; and reorganize spoken thoughts. It does not summarize the transcript or convert spoken instructions into a numbered or bulleted procedure.

Text Polishing is optional and supports `en-US` in Fast, Batch, and Live modes. Set `config.language` to `en-US`, omit `config.language_detection`, and add `outputs.text_polishing` as a top-level field alongside `config`.

## Enable Text Polishing

```json
{
  "file": "https://example.com/audio.mp3",
  "config": {
    "language": "en-US",
    "diarization": true
  },
  "outputs": {
    "text_polishing": {
      "enabled": true
    }
  }
}
```

If `outputs.text_polishing` is omitted or `enabled` is false, Scribe returns only the transcript.

## Fast and Batch Responses

The original transcript remains under `result` and is never overwritten. The polished text is returned under `outputs.text_polishing.text`; each Batch output file carries its own value. Fast responds only after both transcription and polishing complete.

With diarization enabled, polished text uses inline `[Speaker N]` labels. Channel separation uses `[channel0]` and `[channel1]`; when both are enabled, the channel prefix precedes the speaker label. Consecutive sentences from the same speaker share one label, and a new label begins when the speaker changes.

```json
{
  "request_id": "3f2b8c1e-7a4d-4e9b-b6f0-1c5d8a2e9b47",
  "result": {
    "text_display": "Um, welcome everyone, uh, let's get started."
  },
  "outputs": {
    "text_polishing": {
      "text": "[Speaker 1] Welcome everyone. [Speaker 2] Let's get started."
    }
  }
}
```

## Live Behavior

Add the output configuration to `session.update`:

```json
{
  "type": "session.update",
  "config": {
    "language": "en-US",
    "diarization": true
  },
  "audio": {
    "format": "pcm16"
  },
  "outputs": {
    "text_polishing": {
      "enabled": true,
      "include_transcript": true
    }
  }
}
```

Polished results arrive in `transcription.completed` events. One polished event can cover several speech items: it uses the first covered item's `item_id`, its timing spans all covered items, and later covered items do not receive their own completed events. Append `outputs.text_polishing.text` to the document; a later event does not replace earlier text.

`include_transcript` is Live-only and defaults to `false`. When false, the completed event contains polished text but no original `transcript`. Polishing adds latency because the completed event is sent only after polishing finishes.

## Handle Polishing Failures

Transcription still succeeds when polishing fails. Fast remains HTTP `200`, a Batch file remains `FILE_SUCCEEDED`, and a Live session remains open. The `outputs.text_polishing.text` value contains only one of these markers:

| Marker | Cause |
|---|---|
| `[polishing_failed: no_asr_result]` | Empty transcript or only indistinct speech |
| `[polishing_failed: timeout]` | Polishing timed out |
| `[polishing_failed: partial]` | Only part of the text could be polished |
| `[polishing_failed: server_error]` | Polishing service failure |
| `[polishing_failed: error]` | Unknown failure |

Do not display a value beginning with `[polishing_failed:` as polished speech. In Fast and Batch, use `result.text_display`; in Live, use `transcript`, which is present only when `include_transcript` is true.

If the selected locale does not support Text Polishing, Fast and Batch return `400 INVALID_INPUT` without a transcript. Live returns a nonfatal `invalid_config` error and leaves the prior session configuration in effect.

## Sources

- Snapshot 2026-11-19: `developers.zoom.us/docs/ai-services/scribe/text-polishing.md`
- Snapshot 2026-11-19: `developers.zoom.us/docs/ai-services/scribe/live-mode.md`
- OpenAPI snapshot 2026-11-19: `developers.zoom.us/docs/api/ai-services/openapi-ai-services.json`
