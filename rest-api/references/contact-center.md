# Zoom Contact Center API

Omnichannel contact center management API.

## Overview

Zoom Contact Center (ZCC) is a cloud-native omnichannel contact center solution. The API enables management of agents, queues, engagements, flows, skills, and reporting for voice, video, chat, and SMS channels.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with contact center scopes:
- `contact_center:read` - Read contact center data
- `contact_center:write` - Manage contact center
- `contact_center:read:admin` - Admin read access
- `contact_center:write:admin` - Admin write access

## Key Endpoint Categories

### Queues

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/queues` | List queues |
| POST | `/contact_center/queues` | Create queue |
| GET | `/contact_center/queues/{queueId}` | Get queue details |
| PATCH | `/contact_center/queues/{queueId}` | Update queue |
| DELETE | `/contact_center/queues/{queueId}` | Delete queue |
| GET | `/contact_center/queues/{queueId}/agents` | List queue agents |
| POST | `/contact_center/queues/{queueId}/agents` | Add agent to queue |

### Agents

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/agents` | List agents |
| GET | `/contact_center/agents/{agentId}` | Get agent details |
| PATCH | `/contact_center/agents/{agentId}` | Update agent |
| GET | `/contact_center/agents/{agentId}/status` | Get agent status |
| PATCH | `/contact_center/agents/{agentId}/status` | Update agent status |

### Engagements

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/engagements` | List engagements |
| GET | `/contact_center/engagements/{engagementId}` | Get engagement details |
| GET | `/contact_center/engagements/{engagementId}/events` | Get engagement events |

### Voice Calls

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/accounts/{accountId}/contact_center/voice_calls` | List voice call logs |
| GET | `/contact_center/voice_calls/{callId}` | Get call details |
| GET | `/contact_center/voice_calls/{callId}/recording` | Get call recording |

### SMS

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/accounts/{accountId}/contact_center/sms` | List SMS logs |
| GET | `/contact_center/sms/{engagementId}` | Get SMS details |

### Skills

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/skills` | List skills |
| POST | `/contact_center/skills` | Create skill |
| GET | `/contact_center/skills/{skillId}` | Get skill details |
| PATCH | `/contact_center/skills/{skillId}` | Update skill |
| DELETE | `/contact_center/skills/{skillId}` | Delete skill |
| GET | `/contact_center/skills/categories` | List skill categories |

### Flows

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/flows` | List flows |
| POST | `/contact_center/flows` | Create flow |
| GET | `/contact_center/flows/{flowId}` | Get flow details |
| PATCH | `/contact_center/flows/{flowId}` | Update flow |
| DELETE | `/contact_center/flows/{flowId}` | Delete flow |

### Routing Profiles

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/routing_profiles` | List routing profiles |
| POST | `/contact_center/routing_profiles` | Create routing profile |
| GET | `/contact_center/routing_profiles/{profileId}` | Get profile details |

### Reports

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/reports/agent_performance` | Agent performance report |
| GET | `/contact_center/reports/queue_performance` | Queue performance report |
| GET | `/contact_center/reports/engagement_logs` | Engagement logs |
| GET | `/contact_center/reports/historical` | Historical reports |

## Example: List Voice Call Logs

```bash
curl -X GET "https://api.zoom.us/v2/accounts/{accountId}/contact_center/voice_calls?from=2024-01-01&to=2024-01-31&page_size=30" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "page_size": 30,
  "next_page_token": "abc123",
  "voice_calls": [
    {
      "engagement_id": "eng_xyz",
      "call_type": "inbound",
      "direction": "inbound",
      "start_time": "2024-01-15T10:00:00Z",
      "end_time": "2024-01-15T10:05:00Z",
      "duration": 300,
      "agent": {
        "user_id": "agent_abc",
        "display_name": "John Doe"
      },
      "consumer": {
        "phone_number": "+14155551234"
      },
      "queue": {
        "queue_id": "q_support",
        "queue_name": "Support Queue"
      },
      "disposition": "resolved"
    }
  ]
}
```

## Example: Get Agent Status

```bash
curl -X GET "https://api.zoom.us/v2/contact_center/agents/{agentId}/status" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "agent_id": "agent_abc",
  "status": "available",
  "status_since": "2024-01-15T09:00:00Z",
  "channels": {
    "voice": "available",
    "chat": "available",
    "video": "available"
  },
  "current_engagements": 0,
  "max_engagements": 3
}
```

## Example: Create Queue

```bash
curl -X POST "https://api.zoom.us/v2/contact_center/queues" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sales Queue",
    "description": "Queue for sales inquiries",
    "channels": ["voice", "chat"],
    "routing_method": "skills_based",
    "max_wait_time": 300
  }'
```

## Agent Status

| Status | Description |
|--------|-------------|
| `available` | Ready for engagements |
| `busy` | Currently on engagement |
| `away` | Temporarily unavailable |
| `offline` | Logged out |
| `break` | On scheduled break |

## Engagement Types

| Type | Description |
|------|-------------|
| `voice` | Phone call |
| `video` | Video call |
| `chat` | Web/app chat |
| `sms` | SMS message |
| `email` | Email engagement |

## Routing Methods

| Method | Description |
|--------|-------------|
| `round_robin` | Distribute evenly |
| `skills_based` | Route by agent skills |
| `longest_idle` | Route to longest idle agent |
| `priority` | Route by priority |

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/contact-center/methods/
- **Contact Center Documentation**: https://developers.zoom.us/docs/contact-center/
- **ZCC Developer Guide**: https://developers.zoom.us/docs/contact-center/developer-guide/
