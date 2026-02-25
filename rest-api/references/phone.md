# Zoom Phone API

Cloud-based phone system API for voice, SMS, and call management.

## Overview

Zoom Phone is a cloud phone system that integrates voice, video, and messaging. The API enables programmatic control of call logs, call queues, auto receptionists, voicemail, SMS, and user phone settings.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with Zoom Phone scopes:
- `phone:read` - Read phone data
- `phone:write` - Manage phone settings
- `phone:read:admin` - Admin read access
- `phone:write:admin` - Admin write access
- `phone_sms:read` - Read SMS messages
- `phone_sms:write` - Send SMS messages

## Key Endpoint Categories

### Users & Settings

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/users` | List phone users |
| GET | `/phone/users/{userId}` | Get user phone profile |
| PATCH | `/phone/users/{userId}/settings` | Update user settings |
| GET | `/phone/users/{userId}/call_logs` | Get user call logs |

### Call Logs

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/call_logs` | List account call logs |
| GET | `/phone/call_history/{callId}/details` | Get call details |
| GET | `/phone/users/{userId}/recordings` | Get user recordings |

### Call Queues

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/call_queues` | List call queues |
| POST | `/phone/call_queues` | Create call queue |
| GET | `/phone/call_queues/{callQueueId}` | Get queue details |
| PATCH | `/phone/call_queues/{callQueueId}` | Update call queue |
| DELETE | `/phone/call_queues/{callQueueId}` | Delete call queue |
| GET | `/phone/call_queues/{callQueueId}/members` | List queue members |
| POST | `/phone/call_queues/{callQueueId}/members` | Add queue member |

### Auto Receptionists

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/auto_receptionists` | List auto receptionists |
| POST | `/phone/auto_receptionists` | Create auto receptionist |
| GET | `/phone/auto_receptionists/{autoReceptionistId}` | Get details |
| PATCH | `/phone/auto_receptionists/{autoReceptionistId}` | Update settings |

### Phone Numbers

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/numbers` | List phone numbers |
| POST | `/phone/numbers` | Add phone number |
| GET | `/phone/numbers/{numberId}` | Get number details |
| PATCH | `/phone/numbers/{numberId}` | Update number |
| DELETE | `/phone/numbers/{numberId}` | Remove number |

### SMS

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/users/{userId}/sms/sessions` | List SMS sessions |
| GET | `/phone/sms/messages/{messageId}` | Get SMS message |
| POST | `/phone/users/{userId}/sms` | Send SMS |

### Voicemail

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/users/{userId}/voice_mails` | List voicemails |
| GET | `/phone/voice_mails/{voicemailId}` | Get voicemail |
| DELETE | `/phone/voice_mails/{voicemailId}` | Delete voicemail |

### Shared Line Groups

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/shared_line_groups` | List shared line groups |
| POST | `/phone/shared_line_groups` | Create shared line group |
| GET | `/phone/shared_line_groups/{groupId}` | Get group details |
| PATCH | `/phone/shared_line_groups/{groupId}` | Update group |

### Common Areas

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/common_areas` | List common area phones |
| POST | `/phone/common_areas` | Add common area phone |
| GET | `/phone/common_areas/{commonAreaId}` | Get details |

### Emergency Services

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/phone/emergency/addresses` | List emergency addresses |
| POST | `/phone/emergency/addresses` | Add emergency address |
| GET | `/phone/emergency/addresses/{addressId}` | Get address details |

## Example: Get User Call Logs

```bash
curl -X GET "https://api.zoom.us/v2/phone/users/{userId}/call_logs?from=2024-01-01&to=2024-01-31&page_size=30" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "page_size": 30,
  "next_page_token": "abc123",
  "call_logs": [
    {
      "id": "call_xyz",
      "direction": "outbound",
      "duration": 180,
      "start_time": "2024-01-15T10:30:00Z",
      "end_time": "2024-01-15T10:33:00Z",
      "caller_number": "+14155551234",
      "callee_number": "+14155555678",
      "result": "answered",
      "recording_url": "https://zoom.us/recording/abc123"
    }
  ]
}
```

## Example: Create Call Queue

```bash
curl -X POST "https://api.zoom.us/v2/phone/call_queues" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sales Queue",
    "extension_number": "800",
    "site_id": "site_abc",
    "description": "Sales team call queue"
  }'
```

## Example: Send SMS

```bash
curl -X POST "https://api.zoom.us/v2/phone/users/{userId}/sms" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "+14155555678",
    "message": "Hello from Zoom Phone!"
  }'
```

## Call Types

| Type | Description |
|------|-------------|
| `inbound` | Incoming call |
| `outbound` | Outgoing call |
| `missed` | Missed call |
| `voicemail` | Call went to voicemail |

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/phone/methods/
- **Zoom Phone Guide**: https://developers.zoom.us/docs/zoom-phone/
- **Call Handling Guide**: https://developers.zoom.us/docs/zoom-phone/call-handling/
