# Zoom Events API (Webinars Plus & Events)

Large-scale virtual events and conferences API.

## Overview

Zoom Events (formerly Zoom Webinars Plus) enables hosting large-scale virtual and hybrid events with features like multi-session conferences, expo halls, networking, and registration management. The API provides comprehensive event lifecycle management.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with events scopes:
- `event:read` - Read event data
- `event:write` - Create/manage events
- `event:read:admin` - Admin read access
- `event:write:admin` - Admin write access

## Key Endpoint Categories

### Events

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/events` | List events |
| POST | `/events` | Create event |
| GET | `/events/{eventId}` | Get event details |
| PATCH | `/events/{eventId}` | Update event |
| DELETE | `/events/{eventId}` | Delete event |
| POST | `/events/{eventId}/publish` | Publish event |

### Sessions

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/events/{eventId}/sessions` | List sessions |
| POST | `/events/{eventId}/sessions` | Create session |
| GET | `/events/{eventId}/sessions/{sessionId}` | Get session details |
| PATCH | `/events/{eventId}/sessions/{sessionId}` | Update session |
| DELETE | `/events/{eventId}/sessions/{sessionId}` | Delete session |

### Registration

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/events/{eventId}/registrants` | List registrants |
| POST | `/events/{eventId}/registrants` | Add registrant |
| GET | `/events/{eventId}/registrants/{registrantId}` | Get registrant details |
| PATCH | `/events/{eventId}/registrants/{registrantId}` | Update registrant |
| DELETE | `/events/{eventId}/registrants/{registrantId}` | Remove registrant |
| PUT | `/events/{eventId}/registrants/status` | Update registrant status |

### Speakers

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/events/{eventId}/speakers` | List speakers |
| POST | `/events/{eventId}/speakers` | Add speaker |
| GET | `/events/{eventId}/speakers/{speakerId}` | Get speaker details |
| PATCH | `/events/{eventId}/speakers/{speakerId}` | Update speaker |
| DELETE | `/events/{eventId}/speakers/{speakerId}` | Remove speaker |

### Sponsors

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/events/{eventId}/sponsors` | List sponsors |
| POST | `/events/{eventId}/sponsors` | Add sponsor |
| PATCH | `/events/{eventId}/sponsors/{sponsorId}` | Update sponsor |
| DELETE | `/events/{eventId}/sponsors/{sponsorId}` | Remove sponsor |

### Expo & Booths

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/events/{eventId}/expo/booths` | List expo booths |
| POST | `/events/{eventId}/expo/booths` | Create booth |
| PATCH | `/events/{eventId}/expo/booths/{boothId}` | Update booth |

### Analytics

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/events/{eventId}/analytics` | Get event analytics |
| GET | `/events/{eventId}/sessions/{sessionId}/analytics` | Get session analytics |
| GET | `/events/{eventId}/engagement` | Get engagement metrics |

## Example: Create an Event

```bash
curl -X POST "https://api.zoom.us/v2/events" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Annual Tech Conference 2024",
    "description": "Our flagship technology conference",
    "start_time": "2024-03-15T09:00:00Z",
    "end_time": "2024-03-17T18:00:00Z",
    "timezone": "America/Los_Angeles",
    "type": "multi_session",
    "registration_type": "required",
    "settings": {
      "approval_type": "automatic",
      "attendee_limit": 5000,
      "show_social_share_buttons": true
    }
  }'
```

### Response

```json
{
  "id": "evt_abc123",
  "uuid": "xyz789",
  "name": "Annual Tech Conference 2024",
  "start_time": "2024-03-15T09:00:00Z",
  "end_time": "2024-03-17T18:00:00Z",
  "status": "draft",
  "registration_url": "https://events.zoom.us/ev/abc123",
  "hub_url": "https://events.zoom.us/e/abc123"
}
```

## Example: Add Session to Event

```bash
curl -X POST "https://api.zoom.us/v2/events/{eventId}/sessions" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Keynote: Future of AI",
    "description": "Opening keynote presentation",
    "start_time": "2024-03-15T09:00:00Z",
    "duration": 60,
    "type": "webinar",
    "track": "Main Stage",
    "speakers": ["speaker_abc"]
  }'
```

## Example: List Registrants

```bash
curl -X GET "https://api.zoom.us/v2/events/{eventId}/registrants?status=approved&page_size=100" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "page_size": 100,
  "next_page_token": "token123",
  "registrants": [
    {
      "id": "reg_xyz",
      "email": "attendee@example.com",
      "first_name": "Jane",
      "last_name": "Smith",
      "status": "approved",
      "registered_at": "2024-01-15T10:00:00Z",
      "sessions": ["sess_abc", "sess_def"]
    }
  ]
}
```

## Event Types

| Type | Description |
|------|-------------|
| `single_session` | Single webinar/session event |
| `multi_session` | Multi-track conference |
| `series` | Recurring event series |

## Session Types

| Type | Description |
|------|-------------|
| `webinar` | Standard webinar session |
| `meeting` | Interactive meeting session |
| `networking` | Networking session |
| `expo` | Expo/booth session |
| `break` | Break/intermission |

## Registration Status

| Status | Description |
|--------|-------------|
| `pending` | Awaiting approval |
| `approved` | Registration approved |
| `denied` | Registration denied |
| `cancelled` | Registration cancelled |

## Event Status

| Status | Description |
|--------|-------------|
| `draft` | Event in draft mode |
| `published` | Event is live/published |
| `started` | Event has started |
| `ended` | Event has concluded |
| `cancelled` | Event cancelled |

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Events
- **Zoom Events Documentation**: https://developers.zoom.us/docs/zoom-events/
- **Event Planning Guide**: https://developers.zoom.us/docs/zoom-events/planning/
