# Video SDK API

Server-side APIs for Zoom Video SDK session management.

## Overview

The Video SDK API provides server-side endpoints for managing Video SDK sessions, including session creation, user management, recording control, and analytics. These APIs complement the client-side Video SDK.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires Server-to-Server OAuth with Video SDK scopes:
- `videosdk:read` - Read session data
- `videosdk:write` - Create/manage sessions
- `videosdk:read:admin` - Admin access

## Key Endpoints

### Sessions

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/videosdk/sessions` | List sessions |
| GET | `/videosdk/sessions/{sessionId}` | Get session details |
| DELETE | `/videosdk/sessions/{sessionId}` | End session |
| GET | `/videosdk/sessions/{sessionId}/users` | List session participants |

### Session Recording

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/videosdk/sessions/{sessionId}/recording/start` | Start recording |
| POST | `/videosdk/sessions/{sessionId}/recording/stop` | Stop recording |
| GET | `/videosdk/sessions/{sessionId}/recordings` | List recordings |
| DELETE | `/videosdk/sessions/{sessionId}/recordings/{recordingId}` | Delete recording |

### Session Reports

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/videosdk/reports/sessions` | Session usage report |
| GET | `/videosdk/reports/sessions/{sessionId}` | Session detail report |
| GET | `/videosdk/reports/sessions/{sessionId}/participants` | Participant report |

### Quality of Service

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/videosdk/sessions/{sessionId}/qos` | Get session QoS data |
| GET | `/videosdk/sessions/{sessionId}/qos/{participantId}` | Get participant QoS |

## JWT Token Generation

Video SDK requires JWT tokens for client authentication:

```javascript
const jwt = require('jsonwebtoken');

function generateVideoSDKJWT(sessionName, role, userId) {
  const payload = {
    app_key: process.env.VIDEO_SDK_KEY,
    tpc: sessionName,        // Topic (session name)
    role_type: role,         // 0 = participant, 1 = host
    user_identity: userId,   // Unique user identifier
    version: 1,
    iat: Math.floor(Date.now() / 1000),
    exp: Math.floor(Date.now() / 1000) + 3600  // 1 hour
  };
  
  return jwt.sign(payload, process.env.VIDEO_SDK_SECRET);
}
```

### Role Types

| Role | Value | Description |
|------|-------|-------------|
| Participant | 0 | Can join, participate |
| Host | 1 | Full control, can manage |

## Example: List Sessions

```bash
curl "https://api.zoom.us/v2/videosdk/sessions?from=2024-01-01&to=2024-01-31" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "from": "2024-01-01",
  "to": "2024-01-31",
  "page_size": 30,
  "total_records": 150,
  "sessions": [
    {
      "id": "sess_abc123",
      "session_name": "Daily Standup",
      "start_time": "2024-01-15T10:00:00Z",
      "end_time": "2024-01-15T10:30:00Z",
      "duration": 30,
      "participants_count": 8,
      "has_recording": true,
      "status": "ended"
    }
  ]
}
```

## Example: Get Session Details

```bash
curl "https://api.zoom.us/v2/videosdk/sessions/{sessionId}" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "id": "sess_abc123",
  "session_name": "Daily Standup",
  "host_id": "user_xyz",
  "start_time": "2024-01-15T10:00:00Z",
  "end_time": "2024-01-15T10:30:00Z",
  "duration": 30,
  "status": "ended",
  "participants": [
    {
      "id": "part_001",
      "user_id": "user_xyz",
      "user_name": "John Doe",
      "join_time": "2024-01-15T10:00:00Z",
      "leave_time": "2024-01-15T10:30:00Z",
      "duration": 30
    }
  ],
  "recording": {
    "has_recording": true,
    "recording_count": 2
  }
}
```

## Example: Start Cloud Recording

```bash
curl -X POST "https://api.zoom.us/v2/videosdk/sessions/{sessionId}/recording/start" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "recording_type": "cloud"
  }'
```

## Example: Get Session QoS

```bash
curl "https://api.zoom.us/v2/videosdk/sessions/{sessionId}/qos" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "session_id": "sess_abc123",
  "qos_data": {
    "audio": {
      "bitrate": "128kbps",
      "latency": "45ms",
      "jitter": "12ms",
      "packet_loss": "0.1%"
    },
    "video": {
      "bitrate": "2.5mbps",
      "resolution": "1920x1080",
      "frame_rate": "30fps",
      "latency": "65ms",
      "packet_loss": "0.2%"
    },
    "screen_share": {
      "bitrate": "3.0mbps",
      "resolution": "1920x1080",
      "frame_rate": "15fps"
    }
  }
}
```

## Usage Reports

### Get Usage Summary

```bash
curl "https://api.zoom.us/v2/videosdk/reports/sessions?from=2024-01-01&to=2024-01-31&type=summary" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "from": "2024-01-01",
  "to": "2024-01-31",
  "summary": {
    "total_sessions": 500,
    "total_participants": 2500,
    "total_minutes": 15000,
    "unique_hosts": 50,
    "recording_minutes": 5000,
    "average_session_duration": 30
  }
}
```

## Session Management via SDK

While the API handles server-side operations, client-side session control uses the Video SDK:

```javascript
// Client-side (complements server API)
import ZoomVideo from '@zoom/videosdk';

const client = ZoomVideo.createClient();

await client.init('en-US', 'CDN');

await client.join(sessionName, jwt, userName, password);

// Session operations
const stream = client.getMediaStream();
await stream.startVideo();
await stream.startAudio();
```

## Webhooks

| Event | Description |
|-------|-------------|
| `session.started` | Session began |
| `session.ended` | Session ended |
| `session.participant_joined` | User joined |
| `session.participant_left` | User left |
| `session.recording_started` | Recording began |
| `session.recording_completed` | Recording ready |

## Error Codes

| Code | Description |
|------|-------------|
| 3001 | Session not found |
| 3002 | Session already ended |
| 3003 | User not in session |
| 3004 | Recording not enabled |
| 3005 | Invalid JWT token |

## Best Practices

1. **Generate unique session names** - Avoid conflicts
2. **Set appropriate JWT expiry** - Balance security and usability
3. **Monitor QoS metrics** - Track quality issues
4. **Use webhooks for events** - Don't poll for status
5. **Implement retry logic** - Handle transient failures

## Resources

- **Video SDK Overview**: https://developers.zoom.us/docs/video-sdk/
- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-video-sdk-api/methods/
- **JWT Guide**: https://developers.zoom.us/docs/video-sdk/auth/
- **Sample Apps**: https://github.com/zoom/videosdk-web-sample
