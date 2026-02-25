# Zoom Rooms API

Manage Zoom Rooms devices, settings, and scheduling.

## Overview

Zoom Rooms are conference room systems. The API allows you to:
- Monitor room status and health
- Manage room settings
- Schedule meetings for rooms
- Control devices remotely
- Get usage reports

## Base URL

```
https://api.zoom.us/v2/rooms
```

## Authentication

Requires OAuth 2.0 with admin scopes. See `general/references/authentication.md`.

## Core Endpoints

### Rooms

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rooms` | List all Zoom Rooms |
| GET | `/rooms/{roomId}` | Get room details |
| PATCH | `/rooms/{roomId}` | Update room |
| DELETE | `/rooms/{roomId}` | Delete room |
| GET | `/rooms/{roomId}/settings` | Get room settings |
| PATCH | `/rooms/{roomId}/settings` | Update settings |

### Devices

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rooms/{roomId}/devices` | List devices in room |
| PATCH | `/rooms/{roomId}/devices/{deviceId}` | Update device |

### Locations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rooms/locations` | List locations |
| POST | `/rooms/locations` | Create location |
| GET | `/rooms/locations/{locationId}` | Get location |
| PATCH | `/rooms/locations/{locationId}` | Update location |
| DELETE | `/rooms/locations/{locationId}` | Delete location |

### Meetings & Scheduling

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rooms/{roomId}/meetings` | List room meetings |
| GET | `/rooms/{roomId}/events` | Get room calendar |

## Common Operations

### List All Rooms

```javascript
// GET /rooms
const params = new URLSearchParams({
  page_size: 30,
  page_number: 1,
  status: 'Available'  // Optional filter
});

const response = await fetch(
  `https://api.zoom.us/v2/rooms?${params}`,
  {
    headers: { 'Authorization': `Bearer ${accessToken}` }
  }
);

const rooms = await response.json();
// {
//   "rooms": [
//     {
//       "id": "room_abc123",
//       "name": "Conference Room A",
//       "room_id": "abc123",
//       "location_id": "loc_001",
//       "status": "Available",
//       "health": "healthy",
//       "device_ip": "192.168.1.100"
//     }
//   ],
//   "page_count": 5,
//   "page_number": 1,
//   "page_size": 30,
//   "total_records": 142
// }
```

### Get Room Status

```javascript
// GET /rooms/{roomId}
const room = await fetch(
  `https://api.zoom.us/v2/rooms/${roomId}`,
  {
    headers: { 'Authorization': `Bearer ${accessToken}` }
  }
).then(r => r.json());

// {
//   "id": "room_abc123",
//   "name": "Conference Room A",
//   "status": "InMeeting",  // Available, InMeeting, Offline, UnderConstruction
//   "health": "healthy",    // healthy, warning, critical
//   "issues": [],
//   "device_ip": "192.168.1.100",
//   "activation_code": "123456",
//   "room_id": "abc123",
//   "location_id": "loc_001"
// }
```

### Update Room Settings

```javascript
// PATCH /rooms/{roomId}/settings
await fetch(
  `https://api.zoom.us/v2/rooms/${roomId}/settings`,
  {
    method: 'PATCH',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      meeting_security: {
        auto_security: true,
        encryption_type: 'enhanced_encryption'
      },
      notification: {
        upcoming_meeting_alert: true,
        alert_before_start: 5  // minutes
      },
      zoom_rooms: {
        auto_start_scheduled_meeting: true,
        auto_stop_scheduled_meeting: true,
        upcoming_meeting_alert: 10  // minutes
      }
    })
  }
);
```

### Get Room Calendar

```javascript
// GET /rooms/{roomId}/events
const params = new URLSearchParams({
  from: '2024-01-15',
  to: '2024-01-15'
});

const events = await fetch(
  `https://api.zoom.us/v2/rooms/${roomId}/events?${params}`,
  {
    headers: { 'Authorization': `Bearer ${accessToken}` }
  }
).then(r => r.json());

// {
//   "events": [
//     {
//       "id": "event_001",
//       "meeting_id": "12345678901",
//       "topic": "Team Standup",
//       "start_time": "2024-01-15T09:00:00Z",
//       "end_time": "2024-01-15T09:30:00Z",
//       "host": "john@example.com"
//     }
//   ]
// }
```

### Create Location Hierarchy

```javascript
// POST /rooms/locations
// Create building
const building = await fetch(
  'https://api.zoom.us/v2/rooms/locations',
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      name: 'HQ Building',
      parent_location_id: '',  // Top-level
      type: 'building',
      address: '123 Main St, San Francisco, CA'
    })
  }
).then(r => r.json());

// Create floor
const floor = await fetch(
  'https://api.zoom.us/v2/rooms/locations',
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      name: 'Floor 3',
      parent_location_id: building.id,
      type: 'floor'
    })
  }
).then(r => r.json());
```

## Room Status

| Status | Description |
|--------|-------------|
| `Available` | Room is free |
| `InMeeting` | Meeting in progress |
| `Offline` | Device is offline |
| `UnderConstruction` | Room is being set up |

## Health Status

| Health | Description |
|--------|-------------|
| `healthy` | All systems normal |
| `warning` | Minor issues detected |
| `critical` | Major issues, needs attention |

### Health Issues

```javascript
// Check room issues
const room = await fetch(`https://api.zoom.us/v2/rooms/${roomId}`)
  .then(r => r.json());

if (room.health !== 'healthy') {
  console.log('Issues:', room.issues);
  // [
  //   { "issue": "low_bandwidth", "severity": "warning" },
  //   { "issue": "microphone_not_working", "severity": "critical" }
  // ]
}
```

## Device Management

### List Devices

```javascript
// GET /rooms/{roomId}/devices
const devices = await fetch(
  `https://api.zoom.us/v2/rooms/${roomId}/devices`,
  {
    headers: { 'Authorization': `Bearer ${accessToken}` }
  }
).then(r => r.json());

// {
//   "devices": [
//     {
//       "id": "dev_001",
//       "device_type": "Controller",
//       "status": "Online",
//       "app_version": "5.15.0"
//     },
//     {
//       "id": "dev_002",
//       "device_type": "SchedulingDisplay",
//       "status": "Online",
//       "app_version": "5.15.0"
//     }
//   ]
// }
```

### Device Types

| Type | Description |
|------|-------------|
| `Controller` | Touch panel controller |
| `SchedulingDisplay` | Outside-room display |
| `Computer` | Room computer |
| `Camera` | Video camera |
| `Speaker` | Audio speaker |
| `Microphone` | Audio microphone |

## Reports

### Room Usage Report

```javascript
// GET /report/rooms
const params = new URLSearchParams({
  from: '2024-01-01',
  to: '2024-01-31'
});

const report = await fetch(
  `https://api.zoom.us/v2/report/rooms?${params}`,
  {
    headers: { 'Authorization': `Bearer ${accessToken}` }
  }
).then(r => r.json());

// Usage statistics per room
```

### Issues Report

```javascript
// GET /rooms/events/zr_issues
const issues = await fetch(
  `https://api.zoom.us/v2/rooms/events/zr_issues?from=2024-01-01&to=2024-01-31`,
  {
    headers: { 'Authorization': `Bearer ${accessToken}` }
  }
).then(r => r.json());
```

## Webhooks

| Event | Trigger |
|-------|---------|
| `zoomroom.activated` | Room activated |
| `zoomroom.started_meeting` | Meeting started |
| `zoomroom.ended_meeting` | Meeting ended |
| `zoomroom.sensor_data` | Sensor data updated |
| `zoomroom.alert` | Health alert triggered |
| `zoomroom.offline` | Room went offline |
| `zoomroom.online` | Room came online |

### Webhook Payload

```json
{
  "event": "zoomroom.alert",
  "payload": {
    "account_id": "abc123",
    "object": {
      "id": "room_xyz",
      "room_name": "Conference Room A",
      "issue": "microphone_not_working",
      "severity": "critical",
      "time": "2024-01-15T10:30:00Z"
    }
  }
}
```

## Required Scopes

| Scope | Description |
|-------|-------------|
| `room:read:admin` | Read room information |
| `room:write:admin` | Manage rooms |
| `dashboard_zr:read:admin` | Room dashboard/reports |

## Use Cases

### Room Availability Dashboard

```javascript
async function getRoomAvailability() {
  const rooms = await fetch('https://api.zoom.us/v2/rooms', {
    headers: { 'Authorization': `Bearer ${token}` }
  }).then(r => r.json());
  
  return rooms.rooms.map(room => ({
    name: room.name,
    available: room.status === 'Available',
    health: room.health
  }));
}
```

### Health Monitoring Alert

```javascript
// Webhook handler
app.post('/webhook/rooms', (req, res) => {
  const { event, payload } = req.body;
  
  if (event === 'zoomroom.alert') {
    const { room_name, issue, severity } = payload.object;
    
    if (severity === 'critical') {
      sendSlackAlert(`Critical issue in ${room_name}: ${issue}`);
    }
  }
  
  res.sendStatus(200);
});
```

### Capacity Planning

```javascript
async function analyzeRoomUsage(from, to) {
  const report = await fetch(
    `https://api.zoom.us/v2/report/rooms?from=${from}&to=${to}`,
    { headers: { 'Authorization': `Bearer ${token}` } }
  ).then(r => r.json());
  
  // Calculate utilization per room
  return report.rooms.map(room => ({
    name: room.name,
    utilization: (room.meeting_minutes / room.available_minutes) * 100
  }));
}
```

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Zoom-Rooms
- **Zoom Rooms**: https://support.zoom.com/hc/en/zoom-rooms
