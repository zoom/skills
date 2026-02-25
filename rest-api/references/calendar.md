# Zoom Calendar API

Manage calendar events, secondary calendars, and scheduling via REST API.

## Overview

Zoom Calendar Service is a fully integrated calendar built into Zoom Workplace. The API allows you to:
- Create and manage calendar events
- Manage secondary/shared calendars
- Configure access controls and permissions
- Integrate with external calendar providers

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

OAuth 2.0 (Server-to-Server or User OAuth). See `general/references/authentication.md`.

## Key Features

| Feature | Description |
|---------|-------------|
| **External Booking** | Allow external contacts to schedule appointments |
| **Shared Calendars** | Create team calendars for collaboration |
| **Resource Calendars** | Manage conference rooms, workspaces |
| **Delegate Scheduling** | Schedule on behalf of others |

## Core Endpoints

### Events

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/calendars/{calendarId}/events` | List events |
| POST | `/calendars/{calendarId}/events` | Create event |
| GET | `/calendars/{calendarId}/events/{eventId}` | Get event |
| PATCH | `/calendars/{calendarId}/events/{eventId}` | Update event |
| DELETE | `/calendars/{calendarId}/events/{eventId}` | Delete event |

### Calendars

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/users/{userId}/calendars` | List user's calendars |
| POST | `/users/{userId}/calendars` | Create secondary calendar |
| GET | `/calendars/{calendarId}` | Get calendar details |
| PATCH | `/calendars/{calendarId}` | Update calendar |
| DELETE | `/calendars/{calendarId}` | Delete calendar |

### Access Control

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/calendars/{calendarId}/access` | Get access list |
| POST | `/calendars/{calendarId}/access` | Grant access |
| DELETE | `/calendars/{calendarId}/access/{userId}` | Revoke access |

## Common Operations

### Create Calendar Event

```javascript
async function createCalendarEvent(calendarId, event) {
  const response = await fetch(
    `https://api.zoom.us/v2/calendars/${calendarId}/events`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        summary: event.title,
        description: event.description,
        start: {
          dateTime: event.startTime,  // ISO 8601
          timeZone: event.timeZone
        },
        end: {
          dateTime: event.endTime,
          timeZone: event.timeZone
        },
        attendees: event.attendees.map(email => ({ email })),
        location: event.location,
        reminders: {
          useDefault: false,
          overrides: [
            { method: 'popup', minutes: 15 }
          ]
        }
      })
    }
  );
  
  return response.json();
}

// Usage
await createCalendarEvent('primary', {
  title: 'Team Standup',
  description: 'Daily standup meeting',
  startTime: '2024-01-15T09:00:00',
  endTime: '2024-01-15T09:30:00',
  timeZone: 'America/Los_Angeles',
  attendees: ['john@example.com', 'sarah@example.com'],
  location: 'Zoom Meeting'
});
```

### List Events

```javascript
async function listEvents(calendarId, from, to) {
  const params = new URLSearchParams({
    time_min: from,  // ISO 8601
    time_max: to,
    page_size: 50
  });
  
  const response = await fetch(
    `https://api.zoom.us/v2/calendars/${calendarId}/events?${params}`,
    {
      headers: { 'Authorization': `Bearer ${accessToken}` }
    }
  );
  
  return response.json();
}

// Get events for January 2024
const events = await listEvents(
  'primary',
  '2024-01-01T00:00:00Z',
  '2024-01-31T23:59:59Z'
);
```

### Create Secondary Calendar

```javascript
async function createSecondaryCalendar(userId, name, description) {
  const response = await fetch(
    `https://api.zoom.us/v2/users/${userId}/calendars`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        name: name,
        description: description,
        color: '#4285F4',  // Optional: calendar color
        timezone: 'America/Los_Angeles'
      })
    }
  );
  
  return response.json();
}

// Create a team calendar
await createSecondaryCalendar('me', 'Engineering Team', 'Shared engineering calendar');
```

### Share Calendar

```javascript
async function shareCalendar(calendarId, email, role) {
  const response = await fetch(
    `https://api.zoom.us/v2/calendars/${calendarId}/access`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        email: email,
        role: role  // 'reader', 'writer', 'owner'
      })
    }
  );
  
  return response.json();
}

// Share calendar with a colleague
await shareCalendar('calendar_abc123', 'colleague@example.com', 'writer');
```

## Calendar Integration

### Bi-Directional Sync

Zoom Calendar supports bi-directional sync with:
- **Google Calendar** - OAuth-based, automatic sync
- **Microsoft 365 / Outlook** - Graph API integration

Sync 2.0 features:
- Automatically enabled by default
- Stores up to 24 months of data (6 months past, 18 months future)
- Real-time synchronization via webhooks

### Data Synced

| Field | Description |
|-------|-------------|
| `summary` | Event title |
| `description` | Event description |
| `location` | Event location |
| `attendees` | Participant list |
| `organizer` | Event organizer |
| `iCalUID` | Unique calendar ID |

### Security

- Tokens encrypted at rest (256-bit AES-GCM)
- TLS 1.2 encryption in transit
- OAuth access tokens: 1 hour expiry
- OAuth refresh tokens: 90 days expiry

## Create Event with Zoom Meeting

```javascript
async function createEventWithZoomMeeting(calendarId, event) {
  // 1. Create Zoom meeting first
  const meetingResponse = await fetch(
    'https://api.zoom.us/v2/users/me/meetings',
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        topic: event.title,
        type: 2,  // Scheduled meeting
        start_time: event.startTime,
        duration: event.duration,
        timezone: event.timeZone
      })
    }
  );
  
  const meeting = await meetingResponse.json();
  
  // 2. Create calendar event with meeting link
  const eventResponse = await fetch(
    `https://api.zoom.us/v2/calendars/${calendarId}/events`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        summary: event.title,
        description: `${event.description}\n\nJoin Zoom Meeting:\n${meeting.join_url}`,
        start: {
          dateTime: event.startTime,
          timeZone: event.timeZone
        },
        end: {
          dateTime: event.endTime,
          timeZone: event.timeZone
        },
        attendees: event.attendees.map(email => ({ email })),
        location: meeting.join_url,
        conferenceData: {
          conferenceId: meeting.id.toString(),
          conferenceSolution: {
            name: 'Zoom Meeting'
          },
          entryPoints: [
            {
              entryPointType: 'video',
              uri: meeting.join_url
            }
          ]
        }
      })
    }
  );
  
  return {
    event: await eventResponse.json(),
    meeting: meeting
  };
}
```

## Use Cases

| Use Case | Description |
|----------|-------------|
| **Healthcare** | Integrate appointments into patient management systems |
| **Education** | Manage virtual classes, office hours, tutoring |
| **Event Management** | Virtual event scheduling and management |
| **Sales** | Schedule demos and customer meetings |
| **HR** | Interview scheduling and onboarding |

## Required Scopes

| Scope | Description |
|-------|-------------|
| `calendar:read` | Read calendar data |
| `calendar:write` | Create/update calendar events |
| `calendar:read:admin` | Account-wide read access |
| `calendar:write:admin` | Account-wide write access |

## Webhooks

| Event | Trigger |
|-------|---------|
| `calendar.event_created` | New event created |
| `calendar.event_updated` | Event modified |
| `calendar.event_deleted` | Event deleted |
| `calendar.event_reminder` | Event reminder triggered |

## Resources

- **Calendar API Docs**: https://developers.zoom.us/docs/api/rest/zoom-calendar/
- **Calendar Service Overview**: https://developers.zoom.us/docs/zoom-calendar/
- **Postman Collection**: https://www.postman.com/zoom-developer
