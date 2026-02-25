# REST API - Meetings

Meeting management endpoints.

## Overview

Create, read, update, and delete Zoom meetings programmatically.

## Base URL

```
https://api.zoom.us/v2
```

## Endpoints

### Create Meeting

```bash
POST /users/{userId}/meetings
```

```json
{
  "topic": "My Meeting",
  "type": 2,
  "start_time": "2024-01-15T10:00:00Z",
  "duration": 60,
  "timezone": "America/Los_Angeles",
  "settings": {
    "host_video": true,
    "participant_video": true,
    "join_before_host": false
  }
}
```

### Get Meeting

```bash
GET /meetings/{meetingId}
```

### Update Meeting

```bash
PATCH /meetings/{meetingId}
```

### Delete Meeting

```bash
DELETE /meetings/{meetingId}
```

### List Meetings

```bash
GET /users/{userId}/meetings
```

## Meeting Types

| Type | Value | Description |
|------|-------|-------------|
| Instant | 1 | Start immediately |
| Scheduled | 2 | Scheduled meeting |
| Recurring (no fixed time) | 3 | Recurring, no fixed time |
| Recurring (fixed time) | 8 | Recurring with schedule |

## Required Scopes

- `meeting:read` - View meetings
- `meeting:write` - Create/update/delete meetings

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Meetings
