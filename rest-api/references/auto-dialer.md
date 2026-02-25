# Auto Dialer API

Outbound calling campaigns and power dialer functionality for Zoom Contact Center.

## Overview

Auto Dialer APIs enable contact centers to manage outbound calling campaigns, configure dialing strategies, and track campaign performance. These tools automate outbound calling processes for sales, collections, and customer outreach.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with Contact Center scopes:
- `contact_center:read:auto_dialer` - Read campaign data
- `contact_center:write:auto_dialer` - Create/manage campaigns
- `contact_center:read:auto_dialer:admin` - Admin access

## Key Endpoints

### Campaigns

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/auto_dialer/campaigns` | List campaigns |
| POST | `/contact_center/auto_dialer/campaigns` | Create campaign |
| GET | `/contact_center/auto_dialer/campaigns/{campaignId}` | Get campaign |
| PATCH | `/contact_center/auto_dialer/campaigns/{campaignId}` | Update campaign |
| DELETE | `/contact_center/auto_dialer/campaigns/{campaignId}` | Delete campaign |
| POST | `/contact_center/auto_dialer/campaigns/{campaignId}/start` | Start campaign |
| POST | `/contact_center/auto_dialer/campaigns/{campaignId}/pause` | Pause campaign |
| POST | `/contact_center/auto_dialer/campaigns/{campaignId}/stop` | Stop campaign |

### Contact Lists

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/auto_dialer/contact_lists` | List contact lists |
| POST | `/contact_center/auto_dialer/contact_lists` | Create contact list |
| POST | `/contact_center/auto_dialer/contact_lists/{listId}/contacts` | Add contacts |
| DELETE | `/contact_center/auto_dialer/contact_lists/{listId}/contacts` | Remove contacts |
| POST | `/contact_center/auto_dialer/contact_lists/{listId}/import` | Import contacts |

### Campaign Analytics

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/auto_dialer/campaigns/{campaignId}/stats` | Get campaign stats |
| GET | `/contact_center/auto_dialer/campaigns/{campaignId}/calls` | List campaign calls |
| GET | `/contact_center/auto_dialer/reports/summary` | Get summary report |

### DNC (Do Not Call)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/auto_dialer/dnc` | List DNC entries |
| POST | `/contact_center/auto_dialer/dnc` | Add to DNC list |
| DELETE | `/contact_center/auto_dialer/dnc/{phoneNumber}` | Remove from DNC |
| POST | `/contact_center/auto_dialer/dnc/import` | Import DNC list |

## Example: Create Campaign

```bash
curl -X POST "https://api.zoom.us/v2/contact_center/auto_dialer/campaigns" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Q1 Sales Outreach",
    "description": "Quarterly sales campaign",
    "dialing_mode": "preview",
    "contact_list_id": "list_abc123",
    "caller_id": "+14155551234",
    "schedule": {
      "timezone": "America/Los_Angeles",
      "days": ["monday", "tuesday", "wednesday", "thursday", "friday"],
      "start_time": "09:00",
      "end_time": "17:00"
    },
    "settings": {
      "max_attempts": 3,
      "retry_interval_minutes": 60,
      "answering_machine_detection": true,
      "leave_voicemail": true,
      "voicemail_message_id": "vm_123"
    },
    "agent_settings": {
      "skill_group_id": "sg_sales",
      "wrap_up_time_seconds": 30
    }
  }'
```

### Response

```json
{
  "id": "camp_xyz789",
  "name": "Q1 Sales Outreach",
  "status": "draft",
  "dialing_mode": "preview",
  "contact_list_id": "list_abc123",
  "contacts_total": 5000,
  "contacts_remaining": 5000,
  "created_at": "2024-01-25T10:00:00Z"
}
```

## Dialing Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `preview` | Agent sees contact before call | Complex sales |
| `progressive` | Auto-dials when agent available | Moderate volume |
| `power` | Multiple lines per agent | High volume |
| `predictive` | Algorithm-based dialing | Maximum efficiency |

## Campaign Status

| Status | Description |
|--------|-------------|
| `draft` | Not yet started |
| `running` | Actively dialing |
| `paused` | Temporarily stopped |
| `completed` | All contacts processed |
| `stopped` | Manually stopped |

## Example: Import Contacts

```bash
curl -X POST "https://api.zoom.us/v2/contact_center/auto_dialer/contact_lists/{listId}/import" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "contacts": [
      {
        "phone_number": "+14155551111",
        "first_name": "John",
        "last_name": "Doe",
        "email": "john@example.com",
        "custom_fields": {
          "account_id": "ACC001",
          "priority": "high"
        }
      },
      {
        "phone_number": "+14155552222",
        "first_name": "Jane",
        "last_name": "Smith",
        "custom_fields": {
          "account_id": "ACC002",
          "priority": "medium"
        }
      }
    ],
    "deduplicate": true,
    "check_dnc": true
  }'
```

## Campaign Statistics

```bash
curl "https://api.zoom.us/v2/contact_center/auto_dialer/campaigns/{campaignId}/stats" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "campaign_id": "camp_xyz789",
  "stats": {
    "total_contacts": 5000,
    "contacts_dialed": 3500,
    "contacts_remaining": 1500,
    "calls_connected": 2100,
    "calls_answered_live": 1800,
    "calls_voicemail": 300,
    "calls_no_answer": 800,
    "calls_busy": 200,
    "calls_failed": 400,
    "average_talk_time_seconds": 180,
    "total_talk_time_seconds": 324000,
    "connection_rate": 60.0,
    "conversion_rate": 15.5
  },
  "disposition_summary": {
    "interested": 500,
    "callback_requested": 300,
    "not_interested": 800,
    "wrong_number": 100,
    "dnc_requested": 100
  }
}
```

## Call Dispositions

| Disposition | Description |
|-------------|-------------|
| `connected` | Live conversation |
| `voicemail` | Left voicemail |
| `no_answer` | No answer |
| `busy` | Line busy |
| `failed` | Call failed |
| `dnc` | Do not call requested |
| `callback` | Callback scheduled |
| `converted` | Successful outcome |

## Compliance Settings

```json
{
  "compliance": {
    "tcpa_compliant": true,
    "check_dnc_federal": true,
    "check_dnc_state": true,
    "check_dnc_internal": true,
    "max_call_attempts": 3,
    "time_between_attempts_hours": 24,
    "calling_hours": {
      "start": "08:00",
      "end": "21:00",
      "timezone": "local"
    },
    "abandoned_call_threshold_percent": 3
  }
}
```

## Webhooks

| Event | Description |
|-------|-------------|
| `auto_dialer.campaign.started` | Campaign started |
| `auto_dialer.campaign.completed` | Campaign finished |
| `auto_dialer.call.connected` | Call connected |
| `auto_dialer.call.completed` | Call ended |
| `auto_dialer.contact.exhausted` | All attempts used |

## Best Practices

1. **Respect calling hours** - Configure timezone-aware schedules
2. **Maintain DNC lists** - Honor opt-out requests immediately
3. **Monitor abandonment rate** - Stay under regulatory limits
4. **Use appropriate dialing mode** - Match mode to campaign type
5. **Implement dispositions** - Track outcomes for optimization
6. **Test before launch** - Run small pilot campaigns first

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/contact-center/methods/
- **Contact Center Documentation**: https://developers.zoom.us/docs/contact-center/
- **TCPA Compliance**: https://www.fcc.gov/consumers/guides/stop-unwanted-robocalls-and-texts
