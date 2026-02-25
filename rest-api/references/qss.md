# REST API - QSS (Quality of Service Subscription)

Near real-time quality metrics API.

## Overview

QSS provides quality of service telemetry for Zoom Meetings, Webinars, and Phone via webhooks and API endpoints.

## Prerequisites

- QSS add-on subscription
- Business or Enterprise account

## Webhook Events

QSS delivers data via webhook events:

### Event Types

| Event | Description |
|-------|-------------|
| `qss.meeting_participant_qos` | Participant quality metrics |
| `qss.meeting_qos_summary` | Meeting quality summary |
| `qss.phone_call_qos` | Phone call quality metrics |

### Event Payload

```json
{
  "event": "qss.meeting_participant_qos",
  "payload": {
    "account_id": "account_id",
    "object": {
      "meeting_id": "meeting_id",
      "participant_id": "participant_id",
      "user_id": "user_id",
      "metrics": {
        "audio": {
          "bitrate": 64,
          "latency": 50,
          "jitter": 10,
          "packet_loss": 0.5
        },
        "video": {
          "bitrate": 1500,
          "latency": 60,
          "jitter": 15,
          "packet_loss": 0.2,
          "resolution": "1280x720",
          "frame_rate": 30
        },
        "cpu_usage": 25
      },
      "timestamp": "2024-01-15T10:30:00Z"
    }
  }
}
```

## API Endpoints

### Get QSS Settings

```bash
GET /accounts/{accountId}/qss/settings
```

### List QSS Events

```bash
GET /accounts/{accountId}/qss/events
```

Query parameters:
- `from` - Start date
- `to` - End date
- `meeting_id` - Filter by meeting

## Metrics Reference

### Audio Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| `bitrate` | kbps | Audio bitrate |
| `latency` | ms | Round-trip delay |
| `jitter` | ms | Latency variation |
| `packet_loss` | % | Packet loss percentage |

### Video Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| `bitrate` | kbps | Video bitrate |
| `latency` | ms | Round-trip delay |
| `jitter` | ms | Latency variation |
| `packet_loss` | % | Packet loss percentage |
| `resolution` | pixels | Video resolution |
| `frame_rate` | fps | Frames per second |

### System Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| `cpu_usage` | % | Client CPU usage |

## Data Frequency

- Events delivered approximately every minute per participant
- Aggregation may vary by account configuration

## Resources

- **QSS API docs**: https://developers.zoom.us/docs/api/rest/qss-api/
- **QSS API reference**: https://developers.zoom.us/docs/api/rest/reference/qss/methods/
