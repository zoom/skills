# Video Management API

Cloud recording management, video library, and Zoom Clips APIs.

## Overview

Video Management APIs provide access to cloud recordings, the Zoom Clips library, and video asset management. These endpoints enable downloading, sharing, managing, and analyzing recorded video content.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with recording/clips scopes:
- `recording:read` - Read recordings
- `recording:write` - Manage recordings
- `clips:read` - Read clips
- `clips:write` - Create/manage clips

## Cloud Recordings

### Key Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/users/{userId}/recordings` | List user recordings |
| GET | `/meetings/{meetingId}/recordings` | Get meeting recordings |
| DELETE | `/meetings/{meetingId}/recordings` | Delete recordings |
| GET | `/meetings/{meetingId}/recordings/{recordingId}` | Get specific recording |
| DELETE | `/meetings/{meetingId}/recordings/{recordingId}` | Delete specific file |
| PATCH | `/meetings/{meetingId}/recordings/settings` | Update recording settings |
| GET | `/accounts/{accountId}/recordings` | List account recordings |

### Example: List User Recordings

```bash
curl "https://api.zoom.us/v2/users/me/recordings?from=2024-01-01&to=2024-01-31" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "from": "2024-01-01",
  "to": "2024-01-31",
  "page_size": 30,
  "total_records": 5,
  "meetings": [
    {
      "uuid": "abc123==",
      "id": 123456789,
      "topic": "Weekly Team Sync",
      "start_time": "2024-01-15T10:00:00Z",
      "duration": 45,
      "total_size": 156000000,
      "recording_count": 3,
      "recording_files": [
        {
          "id": "rec_1",
          "meeting_id": "abc123==",
          "recording_start": "2024-01-15T10:00:00Z",
          "recording_end": "2024-01-15T10:45:00Z",
          "file_type": "MP4",
          "file_size": 145000000,
          "download_url": "https://zoom.us/rec/download/...",
          "play_url": "https://zoom.us/rec/play/...",
          "status": "completed",
          "recording_type": "shared_screen_with_speaker_view"
        },
        {
          "id": "rec_2",
          "file_type": "M4A",
          "file_size": 8000000,
          "recording_type": "audio_only"
        },
        {
          "id": "rec_3",
          "file_type": "CHAT",
          "file_size": 3000,
          "recording_type": "chat_file"
        }
      ]
    }
  ]
}
```

### Recording Types

| Type | Description |
|------|-------------|
| `shared_screen_with_speaker_view` | Screen share + active speaker |
| `shared_screen_with_gallery_view` | Screen share + gallery |
| `active_speaker` | Active speaker only |
| `gallery_view` | Gallery view |
| `shared_screen` | Screen share only |
| `audio_only` | Audio recording (M4A) |
| `audio_transcript` | Audio transcript (VTT) |
| `chat_file` | Chat messages |
| `timeline` | Timeline file |

## Zoom Clips

### Key Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/clips` | List clips |
| POST | `/clips` | Create clip |
| GET | `/clips/{clipId}` | Get clip details |
| PATCH | `/clips/{clipId}` | Update clip |
| DELETE | `/clips/{clipId}` | Delete clip |
| GET | `/clips/{clipId}/transcript` | Get clip transcript |
| POST | `/clips/{clipId}/share` | Share clip |

### Example: Create Clip from Recording

```bash
curl -X POST "https://api.zoom.us/v2/clips" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "source": {
      "type": "recording",
      "recording_id": "rec_abc123",
      "start_time": 120,
      "end_time": 300
    },
    "title": "Product Demo Highlight",
    "description": "Key features walkthrough"
  }'
```

### Clip Object

```json
{
  "id": "clip_xyz",
  "title": "Product Demo Highlight",
  "description": "Key features walkthrough",
  "duration": 180,
  "status": "ready",
  "created_at": "2024-01-20T15:00:00Z",
  "owner_id": "user_123",
  "play_url": "https://zoom.us/clips/xyz",
  "share_url": "https://zoom.us/clips/share/xyz",
  "thumbnail_url": "https://zoom.us/clips/thumb/xyz",
  "views": 45,
  "transcript_status": "completed"
}
```

## Transcript Management

### Get Recording Transcript

```bash
curl "https://api.zoom.us/v2/meetings/{meetingId}/recordings/{recordingId}/transcript" \
  -H "Authorization: Bearer {accessToken}"
```

### Response (VTT Format)

```
WEBVTT

00:00:00.000 --> 00:00:05.000
Hello everyone, welcome to our weekly sync.

00:00:05.000 --> 00:00:12.000
Today we'll be discussing the Q1 roadmap.
```

## Sharing Settings

```bash
curl -X PATCH "https://api.zoom.us/v2/meetings/{meetingId}/recordings/settings" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "share_recording": "publicly",
    "recording_authentication": false,
    "password": "optional_password",
    "viewer_download": true,
    "approval_type": 0,
    "send_email_to_host": true,
    "show_social_share_buttons": true
  }'
```

### Share Settings Options

| Setting | Values |
|---------|--------|
| `share_recording` | `publicly`, `internally`, `none` |
| `recording_authentication` | `true`, `false` |
| `viewer_download` | `true`, `false` |
| `approval_type` | `0` (auto), `1` (manual), `2` (no registration) |
| `on_demand` | `true`, `false` |

## Download Recordings

```javascript
// Get download URL (valid for limited time)
const recording = await getRecording(meetingId, recordingId);
const downloadUrl = recording.download_url;

// Download with token
const response = await fetch(`${downloadUrl}?access_token=${accessToken}`);
const videoBuffer = await response.arrayBuffer();
```

## Storage Management

### Get Storage Usage

```bash
curl "https://api.zoom.us/v2/users/{userId}/recordings/storage" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "used_storage": 5368709120,
  "total_storage": 10737418240,
  "used_percentage": 50,
  "recordings_count": 150
}
```

## Trash Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/users/{userId}/recordings/trash` | List trashed recordings |
| DELETE | `/users/{userId}/recordings/trash/{meetingId}` | Permanently delete |
| PUT | `/users/{userId}/recordings/trash/{meetingId}/recover` | Recover from trash |

## Webhooks

| Event | Description |
|-------|-------------|
| `recording.completed` | Recording finished processing |
| `recording.transcript_completed` | Transcript ready |
| `recording.deleted` | Recording deleted |
| `recording.trashed` | Recording moved to trash |
| `recording.recovered` | Recording recovered |

## Best Practices

1. **Check recording status** - Wait for `completed` before downloading
2. **Use webhooks** - Don't poll for recording availability
3. **Handle large files** - Stream downloads for large recordings
4. **Respect retention** - Implement automatic cleanup policies
5. **Secure sharing** - Use passwords for sensitive content

## Resources

- **Recording API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Cloud-Recording
- **Clips API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Clips
- **Webhooks**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/events/
