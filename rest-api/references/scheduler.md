# Zoom Scheduler API

Appointment scheduling and booking management API.

## Overview

Zoom Scheduler enables users to share their availability and let others book time with them. The API provides endpoints for managing schedules, availability windows, scheduled events, and routing forms.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with scheduler scopes:
- `scheduler:read` - Read scheduling data
- `scheduler:write` - Create/update schedules
- `scheduler:read:admin` - Admin read access
- `scheduler:write:admin` - Admin write access

## Key Endpoints

### Schedules

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/scheduler/schedules` | List schedules |
| POST | `/scheduler/schedules` | Create schedule |
| GET | `/scheduler/schedules/{scheduleId}` | Get schedule details |
| PATCH | `/scheduler/schedules/{scheduleId}` | Update schedule |
| DELETE | `/scheduler/schedules/{scheduleId}` | Delete schedule |

### Availability

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/scheduler/availability` | Get availability settings |
| POST | `/scheduler/availability` | Create availability window |
| PATCH | `/scheduler/availability` | Update availability |
| DELETE | `/scheduler/availability` | Delete availability |
| GET | `/scheduler/availability/list` | List all availability windows |

### Scheduled Events

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/scheduler/scheduled_events` | Get scheduled event |
| GET | `/scheduler/scheduled_events/list` | List scheduled events |
| PATCH | `/scheduler/scheduled_events` | Update scheduled event |
| DELETE | `/scheduler/scheduled_events` | Cancel scheduled event |
| GET | `/scheduler/scheduled_events/attendee` | Get attendee details |

### Routing Forms

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/scheduler/routing_forms` | List routing forms |
| POST | `/scheduler/routing_forms` | Create routing form |
| GET | `/scheduler/routing_forms/{formId}` | Get form details |
| GET | `/scheduler/routing_forms/response` | Get form responses |

### Analytics

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/scheduler/analytics/report` | Get scheduling analytics |

## Example: Create a Schedule

```bash
curl -X POST "https://api.zoom.us/v2/scheduler/schedules" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "30-Minute Meeting",
    "duration": 30,
    "description": "Quick sync call",
    "location": {
      "type": "zoom_meeting"
    },
    "availability": {
      "timezone": "America/Los_Angeles",
      "windows": [
        {
          "day": "monday",
          "start_time": "09:00",
          "end_time": "17:00"
        },
        {
          "day": "tuesday",
          "start_time": "09:00",
          "end_time": "17:00"
        }
      ]
    }
  }'
```

### Response

```json
{
  "id": "sched_abc123",
  "name": "30-Minute Meeting",
  "duration": 30,
  "booking_url": "https://zoom.us/scheduler/abc123",
  "created_at": "2024-01-15T10:00:00Z"
}
```

## Example: Get Available Slots

```bash
curl -X GET "https://api.zoom.us/v2/scheduler/schedules/{scheduleId}/slots?start_date=2024-01-20&end_date=2024-01-27" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "slots": [
    {
      "start_time": "2024-01-20T09:00:00-08:00",
      "end_time": "2024-01-20T09:30:00-08:00",
      "available": true
    },
    {
      "start_time": "2024-01-20T09:30:00-08:00",
      "end_time": "2024-01-20T10:00:00-08:00",
      "available": true
    }
  ]
}
```

## Example: List Scheduled Events

```bash
curl -X GET "https://api.zoom.us/v2/scheduler/scheduled_events/list?from=2024-01-01&to=2024-01-31" \
  -H "Authorization: Bearer {accessToken}"
```

## Schedule Types

| Type | Description |
|------|-------------|
| `one_on_one` | Individual meeting |
| `group` | Group meeting with multiple invitees |
| `round_robin` | Distributed among team members |
| `collective` | Requires all team members available |

## Location Types

| Type | Description |
|------|-------------|
| `zoom_meeting` | Automatically creates Zoom meeting |
| `phone` | Phone call |
| `in_person` | Physical location |
| `custom` | Custom location/instructions |

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Scheduler
- **Scheduler Documentation**: https://developers.zoom.us/docs/zoom-scheduler/
