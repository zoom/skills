# Zoom Phone Realtime Media Streams

Phone RTMS streams live call audio and transcripts without a bot. It uses the shared RTMS signaling/media model with Phone-specific identity, events, and limits. Sources: archived `developers.zoom.us/docs/rtms/phone/*`, `event-reference/phone-events.md`, and `event-reference/universal-events.md`, captured 2026-11-18.

## Requirements

- Zoom Developer Pack credits; concurrent-stream limits are subscription-based, with a documented default of 2,000 concurrent streams.
- A Zoom Phone license for each enabled user.
- A General OAuth app with the webhook events `phone.call_rtms_started`, `phone.call_rtms_stopped`, and `phone.call_rtms_interrupted`.
- Scopes `phone:read:voice_call_audio` for audio and `phone:read:voice_call_transcript` for transcripts.
- Admin enablement per user under Phone System Management -> user Policy -> In-Call Features -> Realtime Media Streams; select the app and configure automatic start, user controls, and audio notifications.

Zoom recognizes the app for Phone policy configuration only after the required events and scopes are present.

## Connection Sequence

1. `phone.call_rtms_started` supplies `call_id`, `rtms_stream_id`, and `server_urls`. Verify the webhook signature, respond `200` immediately, and persist all three values; the interrupted event does not repeat `server_urls`.
2. Split `server_urls` on commas and select a `wss://` value for the signaling socket.
3. Send signaling handshake `msg_type: 1` with `protocol_version: 2`, `sequence: 0`, `call_id`, `rtms_stream_id`, and signature. Phone requires protocol version 2.
4. The signature is `HMACSHA256(client_id + "," + call_id + "," + rtms_stream_id, client_secret)`, hex encoded. Phone signs `call_id`, not a meeting UUID.
5. On successful signaling response `msg_type: 2`, read `media_server.server_urls.audio` and `.transcript`. Phone provides only audio and transcript.
6. Open one media socket per type. For each, send data handshake `msg_type: 3` with `protocol_version: 2`, the same identity/signature, and `media_type` `1` for audio or `8` for transcript. Do not send a combined mask or the deprecated all-media value.
7. On media response `msg_type: 4` with `status_code: 0`, send client ready ACK `msg_type: 7` with `rtms_stream_id` on the signaling socket. The official Phone quickstart sends only those fields. Zoom's `phone-rtms-auto` sample (`2f0f19b`, 2026-11-18) also sends `media_type`, and its README says to validate that extra field against live calls; use the documented minimal ACK unless live testing proves otherwise.
8. Answer every `msg_type: 12` keep-alive with `msg_type: 13` and the same timestamp, on the socket that received it. Three missed keep-alives interrupt or terminate a connection.
9. On `phone.call_rtms_stopped`, close the signaling and media sockets for that `rtms_stream_id`.

URL fields can contain comma-separated protocol options, including `wss://` and `srt://`. Select the URL for the transport in use; the WebSocket flow uses `wss://`.

## Media

| Media | Handshake `media_type` | Data message | Content |
|---|---|---|---|
| Audio | `1` | `msg_type: 14` | Base64 raw PCM in `content.data`, with `content.channel_id` and `content.timestamp` |
| Transcript | `8` | `msg_type: 17` | JSON text in `content`, attributed by `channel_id` |

Audio is per participant. Keep one capture per `channel_id`, correlate participants from `CALL_INFO_UPDATED`, use timestamps to find gaps, and convert or compress only complete captures.

Treat transcript `msg_type: 17` as a sentence store keyed by `(channel_id, sentence_id)`. `attribute: 1` creates or overwrites; `attribute: 2` replaces the sentence with the complete `data` value and updates `language`, timing, and `timestamp`. `sentence_id` is not unique across channels, `0` is valid, and an update can arrive for a sentence missing after reconnect. Process idempotently and keep the latest version.

## Phone Call Events

The stream can receive `EVENT_UPDATE` call events:

| `event_code` | Name | Meaning |
|---|---|---|
| `3000` | `CALL_INFO_UPDATED` | Participants changed; includes `host_channel_id`, participant names, phone/extension, and `channel_id`. |
| `3001` | `CALL_HOLD` | The RTMS user put the call on hold. |
| `3002` | `CALL_RESUME` | Hold ended. |
| `3003` | `TRANSFER_COMPLETED` | Transfer completed; `transfer_type` is `MEETING`, `EXTENSION`, or `PSTN`. |

Session states are `INACTIVE`, `INITIALIZE`, `STARTED`, `PAUSED`, `RESUMED`, and `STOPPED`.

## Recovery

| Scenario | Trigger | Action |
|---|---|---|
| RTMS server failure | New `phone.call_rtms_started` for an active `rtms_stream_id` | Close everything and reconnect with the new URLs; `call_id` and stream ID stay the same. |
| Signaling dropped | `phone.call_rtms_interrupted` | Close all; after backoff, reconnect signaling and then media from stored URLs. |
| One media socket dropped | `EVENT_UPDATE` `event_code: 2` | Reconnect only that media socket; signaling remains up. |
| Stream terminated | `phone.call_rtms_stopped` `stop_reason: 10` | Restart the full flow if the call is still active. Do not restart for user stop `2`. |

Use exponential backoff starting at 3 seconds, doubling to a 30-second cap; reset the counter after each successful handshake. If retries are exhausted, close all sockets, stop tracking the stream, and surface the failure.

Keep `call_id`, `rtms_stream_id`, the signaling URL, and each media URL for the life of the call. `phone.call_rtms_interrupted` carries no replacement connection details.

## Sources

- `developers.zoom.us/docs/rtms/phone.md`
- `developers.zoom.us/docs/rtms/phone/getting-started.md`
- `developers.zoom.us/docs/rtms/phone/add-features.md`
- `developers.zoom.us/docs/rtms/phone/quickstart.md`
- `developers.zoom.us/docs/rtms/phone/work-with-streams.md`
- `developers.zoom.us/docs/rtms/phone/media.md`
- `developers.zoom.us/docs/rtms/phone/transcript-updates.md`
- `developers.zoom.us/docs/rtms/phone/failover-reconnection.md`
- `developers.zoom.us/docs/rtms/event-reference/phone-events.md`
- `developers.zoom.us/docs/rtms/event-reference/universal-events.md`
- Snapshot provenance: `PROVENANCE-phone-2026-11-18.md`, 2026-11-18T07:13:44Z
