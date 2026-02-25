# RTMS Data Types

Complete reference for all RTMS enums and constants.

## Message Types (RTMS_MESSAGE_TYPE)

| Value | Name | Direction | Description |
|-------|------|-----------|-------------|
| 0 | UNDEFINED | - | Default |
| 1 | SIGNALING_HAND_SHAKE_REQ | Client -> Server | Signaling handshake request |
| 2 | SIGNALING_HAND_SHAKE_RESP | Server -> Client | Signaling handshake response |
| 3 | DATA_HAND_SHAKE_REQ | Client -> Server | Media handshake request |
| 4 | DATA_HAND_SHAKE_RESP | Server -> Client | Media handshake response |
| 5 | EVENT_SUBSCRIPTION | Client -> Server | Subscribe/unsubscribe events |
| 6 | EVENT_UPDATE | Server -> Client | Event notification |
| 7 | CLIENT_READY_ACK | Client -> Server | Ready to receive media |
| 8 | STREAM_STATE_UPDATE | Server -> Client | Stream state changed |
| 9 | SESSION_STATE_UPDATE | Server -> Client | Session state changed |
| 10 | SESSION_STATE_REQ | Client -> Server | Request session state |
| 11 | SESSION_STATE_RESP | Server -> Client | Session state response |
| 12 | KEEP_ALIVE_REQ | Server -> Client | Heartbeat ping |
| 13 | KEEP_ALIVE_RESP | Client -> Server | Heartbeat pong |
| 14 | MEDIA_DATA_AUDIO | Server -> Client | Audio data |
| 15 | MEDIA_DATA_VIDEO | Server -> Client | Video data |
| 16 | MEDIA_DATA_SHARE | Server -> Client | Screen share data |
| 17 | MEDIA_DATA_TRANSCRIPT | Server -> Client | Transcript data |
| 18 | MEDIA_DATA_CHAT | Server -> Client | Chat data |
| 19 | STREAM_STATE_REQ | Client -> Server | Request stream state |
| 20 | STREAM_STATE_RESP | Server -> Client | Stream state response |

## Event Types (RTMS_EVENT_TYPE)

| Value | Name | Description |
|-------|------|-------------|
| 0 | UNDEFINED | Default |
| 1 | FIRST_PACKET_TIMESTAMP | First packet received |
| 2 | ACTIVE_SPEAKER_CHANGE | Active speaker changed |
| 3 | PARTICIPANT_JOIN | Participant joined |
| 4 | PARTICIPANT_LEAVE | Participant left |
| 5 | SHARING_START | Screen sharing started |
| 6 | SHARING_STOP | Screen sharing stopped |
| 7 | MEDIA_CONNECTION_INTERRUPTED | Connection interrupted |

## Status Codes (RTMS_STATUS_CODE)

| Value | Name | Description |
|-------|------|-------------|
| 0 | STATUS_OK | Success |
| 1 | STATUS_CONNECTION_TIMEOUT | Connection timed out |
| 2 | STATUS_INVALID_JSON_MSG_SIZE | Invalid JSON size |
| 3 | STATUS_INVALID_JSON_MSG | Invalid JSON message |
| 4 | STATUS_INVALID_MESSAGE_TYPE | Invalid message type |
| 5 | STATUS_MSG_TYPE_NOT_EXIST | Message type missing |
| 6 | STATUS_MSG_TYPE_NOT_UINT | Message type not integer |
| 7 | STATUS_MEETING_UUID_NOT_EXIST | Meeting UUID missing |
| 8 | STATUS_MEETING_UUID_NOT_STRING | Meeting UUID not string |
| 9 | STATUS_MEETING_UUID_IS_EMPTY | Meeting UUID empty |
| 10 | STATUS_RTMS_STREAM_ID_NOT_EXIST | Stream ID missing |
| 11 | STATUS_RTMS_STREAM_ID_NOT_STRING | Stream ID not string |
| 12 | STATUS_RTMS_STREAM_ID_IS_EMPTY | Stream ID empty |
| 13 | STATUS_SESSION_NOT_FOUND | Session not found |
| 14 | STATUS_SIGNATURE_NOT_EXIST | Signature missing |
| 15 | STATUS_INVALID_SIGNATURE | Invalid signature |
| 16 | STATUS_INVALID_MEETING_OR_STREAM_ID | Invalid meeting/stream |
| 17 | STATUS_DUPLICATE_SIGNAL_REQUEST | Duplicate signaling connection |
| 31 | STATUS_DUPLICATE_MEDIA_DATA_CONNECTION | Duplicate media connection |

## Media Data Types (MEDIA_DATA_TYPE)

Use bitwise OR to combine:

| Value | Name | Constant |
|-------|------|----------|
| 1 | AUDIO | 0x01 |
| 2 | VIDEO | 0x01 << 1 |
| 4 | DESKSHARE | 0x01 << 2 |
| 8 | TRANSCRIPT | 0x01 << 3 |
| 16 | CHAT | 0x01 << 4 |
| 32 | ALL | 0x01 << 5 |

**Common combinations**:
- Audio + Transcript: `1 | 8 = 9`
- Audio + Video: `1 | 2 = 3`
- All media: `32`

## Content Types (MEDIA_CONTENT_TYPE)

| Value | Name | Description |
|-------|------|-------------|
| 0 | UNDEFINED | Default |
| 1 | RTP | Real-time audio/video |
| 2 | RAW_AUDIO | Raw audio data |
| 3 | RAW_VIDEO | Raw video data |
| 4 | FILE_STREAM | File stream |
| 5 | TEXT | Text (transcript, chat) |

## Payload Types / Codecs (MEDIA_PAYLOAD_TYPE)

| Value | Name | Use |
|-------|------|-----|
| 0 | UNDEFINED | - |
| 1 | L16 | Audio - PCM uncompressed |
| 2 | G711 | Audio - compressed |
| 3 | G722 | Audio - compressed |
| 4 | OPUS | Audio - compressed |
| 5 | JPG | Video/Share - when fps <= 5 |
| 6 | PNG | Video/Share - when fps <= 5 |
| 7 | H264 | Video/Share - when fps > 5 |

## Media Data Options (MEDIA_DATA_OPTION)

| Value | Name | Description |
|-------|------|-------------|
| 0 | UNDEFINED | - |
| 1 | AUDIO_MIXED_STREAM | All audio combined |
| 2 | AUDIO_MULTI_STREAMS | Per-participant audio |
| 3 | VIDEO_SINGLE_ACTIVE_STREAM | Active speaker video |
| 4 | VIDEO_MIXED_SPEAKER_VIEW | Speaker view (coming soon) |
| 5 | VIDEO_MIXED_GALLERY_VIEW | Gallery view (coming soon) |

## Media Resolution (MEDIA_RESOLUTION)

| Value | Name | Pixels |
|-------|------|--------|
| 1 | SD | 854x480 or 640x360 |
| 2 | HD | 1280x720 |
| 3 | FHD | 1920x1080 |
| 4 | QHD | 2560x1440 |

## Audio Sample Rate (AUDIO_SAMPLE_RATE)

| Value | Name | Rate |
|-------|------|------|
| 0 | SR_8K | 8000 Hz |
| 1 | SR_16K | 16000 Hz |
| 2 | SR_32K | 32000 Hz |
| 3 | SR_48K | 48000 Hz |

## Audio Channel (AUDIO_CHANNEL)

| Value | Name | Note |
|-------|------|------|
| 1 | MONO | Default |
| 2 | STEREO | Only with OPUS codec! |

## Session State (RTMS_SESSION_STATE)

| Value | Name | Description |
|-------|------|-------------|
| 0 | INACTIVE | Default |
| 1 | INITIALIZE | Session initializing |
| 2 | STARTED | Session started |
| 3 | PAUSED | Session paused |
| 4 | RESUMED | Session resumed |
| 5 | STOPPED | Session stopped |

## Stream State (RTMS_STREAM_STATE)

| Value | Name | Description |
|-------|------|-------------|
| 0 | INACTIVE | Default |
| 1 | ACTIVE | Streaming |
| 2 | INTERRUPTED | Connection problem |
| 3 | TERMINATING | Ending |
| 4 | TERMINATED | Ended |
| 5 | PAUSED | Paused |
| 6 | RESUMED | Resumed |

## Stop Reasons (RTMS_STOP_REASON)

| Value | Name | Description |
|-------|------|-------------|
| 0 | UNDEFINED | Default |
| 1 | STOP_BC_HOST_TRIGGERED | Host stopped |
| 2 | STOP_BC_USER_TRIGGERED | User stopped |
| 3 | STOP_BC_USER_LEFT | User left meeting |
| 4 | STOP_BC_USER_EJECTED | User ejected by host |
| 5 | STOP_BC_APP_DISABLED_BY_HOST | App disabled |
| 6 | STOP_BC_MEETING_ENDED | Meeting ended |
| 7 | STOP_BC_STREAM_CANCELED | Stream canceled |
| 8 | STOP_BC_STREAM_REVOKED | Stream revoked (delete assets!) |
| 9 | STOP_BC_ALL_APPS_DISABLED | All apps disabled |
| 10 | STOP_BC_INTERNAL_EXCEPTION | Internal error |
| 11 | STOP_BC_CONNECTION_TIMEOUT | Connection timeout |
| 12 | STOP_BC_MEETING_CONNECTION_INTERRUPTED | Meeting connection issue |
| 13 | STOP_BC_SIGNAL_CONNECTION_INTERRUPTED | Signaling issue |
| 14 | STOP_BC_DATA_CONNECTION_INTERRUPTED | Data connection issue |
| 15 | STOP_BC_SIGNAL_CONNECTION_CLOSED_ABNORMALLY | Abnormal close |
| 16 | STOP_BC_DATA_CONNECTION_CLOSED_ABNORMALLY | Abnormal close |
| 17 | STOP_BC_EXIT_SIGNAL | Exit signal received |
| 18 | STOP_BC_AUTHENTICATION_FAILURE | Auth failed |

## Transcript Languages (RTMS_TRANSCRIPT_LANGUAGE)

| Value | Name | Language |
|-------|------|----------|
| -1 | LANGUAGE_ID_NONE | None/Auto-detect |
| 0 | LANGUAGE_ID_ARABIC | Arabic |
| 1 | LANGUAGE_ID_BENGALI | Bengali |
| 2 | LANGUAGE_ID_CANTONESE | Cantonese |
| 3 | LANGUAGE_ID_CATALAN | Catalan |
| 4 | LANGUAGE_ID_CHINESE_SIMPLIFIED | Chinese (Simplified) |
| 5 | LANGUAGE_ID_CHINESE_TRADITIONAL | Chinese (Traditional) |
| 6 | LANGUAGE_ID_CZECH | Czech |
| 7 | LANGUAGE_ID_DANISH | Danish |
| 8 | LANGUAGE_ID_DUTCH | Dutch |
| **9** | **LANGUAGE_ID_ENGLISH** | **English (Default)** |
| 10 | LANGUAGE_ID_ESTONIAN | Estonian |
| 11 | LANGUAGE_ID_FINNISH | Finnish |
| 12 | LANGUAGE_ID_FRENCH_CANADA | French (Canada) |
| 13 | LANGUAGE_ID_FRENCH_FRANCE | French (France) |
| 14 | LANGUAGE_ID_GERMAN | German |
| 15 | LANGUAGE_ID_HEBREW | Hebrew |
| 16 | LANGUAGE_ID_HINDI | Hindi |
| 17 | LANGUAGE_ID_HUNGARIAN | Hungarian |
| 18 | LANGUAGE_ID_INDONESIAN | Indonesian |
| 19 | LANGUAGE_ID_ITALIAN | Italian |
| 20 | LANGUAGE_ID_JAPANESE | Japanese |
| 21 | LANGUAGE_ID_KOREAN | Korean |
| 22 | LANGUAGE_ID_MALAY | Malay |
| 23 | LANGUAGE_ID_PERSIAN | Persian |
| 24 | LANGUAGE_ID_POLISH | Polish |
| 25 | LANGUAGE_ID_PORTUGUESE | Portuguese |
| 26 | LANGUAGE_ID_ROMANIAN | Romanian |
| 27 | LANGUAGE_ID_RUSSIAN | Russian |
| 28 | LANGUAGE_ID_SPANISH | Spanish |
| 29 | LANGUAGE_ID_SWEDISH | Swedish |
| 30 | LANGUAGE_ID_TAGALOG | Tagalog |
| 31 | LANGUAGE_ID_TAMIL | Tamil |
| 32 | LANGUAGE_ID_TELUGU | Telugu |
| 33 | LANGUAGE_ID_THAI | Thai |
| 34 | LANGUAGE_ID_TURKISH | Turkish |
| 35 | LANGUAGE_ID_UKRAINIAN | Ukrainian |
| 36 | LANGUAGE_ID_VIETNAMESE | Vietnamese |

## Next Steps

- **[Media Types](media-types.md)** - Parameter configurations
- **[Connection](connection.md)** - Protocol details
- **[Manual WebSocket](../examples/manual-websocket.md)** - Implementation
