# Version Drift

- In `2.5.10`, use the newly named playback-volume APIs when available and treat older volume
  controls as deprecated.
- QoS payloads changed to a richer Web-aligned structure; update parsers for codec, network,
  jitter, packet-loss, and encode/decode metrics rather than assuming the older shape.

Flutter wrapper, native SDKs, and docs can drift independently.

## Upgrade checklist

1. Compare wrapper version metadata and changelog with actual package content.
2. Re-validate API enum names and helper method signatures.
3. Re-test lifecycle: init -> join -> media -> leave -> cleanup.
4. Re-check event constants and error handling mappings.
5. Re-run smoke tests for chat/share/recording/transcription if used.
