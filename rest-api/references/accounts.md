# Accounts API

Account management, sub-accounts, and organization settings.

## Overview

The Accounts API provides endpoints for managing Zoom accounts, including account settings, sub-account management for ISVs and resellers, account plans, and organizational configurations.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with account scopes:
- `account:read` - Read account data
- `account:write` - Modify account settings
- `account:read:admin` - Admin read access
- `account:write:admin` - Admin write access

## Key Endpoints

### Account Information

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/accounts/{accountId}` | Get account details |
| GET | `/accounts/{accountId}/settings` | Get account settings |
| PATCH | `/accounts/{accountId}/settings` | Update account settings |
| GET | `/accounts/{accountId}/lock_settings` | Get locked settings |
| PATCH | `/accounts/{accountId}/lock_settings` | Lock/unlock settings |

### Sub-Accounts (Master Account)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/accounts` | List sub-accounts |
| POST | `/accounts` | Create sub-account |
| DELETE | `/accounts/{accountId}` | Delete sub-account |
| PATCH | `/accounts/{accountId}/options` | Update sub-account options |
| GET | `/accounts/{accountId}/billing` | Get billing info |

### Account Plans

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/accounts/{accountId}/plans` | Get account plans |
| POST | `/accounts/{accountId}/plans` | Subscribe to plan |
| PUT | `/accounts/{accountId}/plans/base` | Update base plan |
| POST | `/accounts/{accountId}/plans/addons` | Add plan addons |

### Account Branding

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/accounts/{accountId}/branding` | Get branding settings |
| PATCH | `/accounts/{accountId}/branding` | Update branding |
| POST | `/accounts/{accountId}/branding/logo` | Upload logo |

## Example: Get Account Details

```bash
curl "https://api.zoom.us/v2/accounts/me" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "id": "abc123xyz",
  "owner_id": "user_owner",
  "owner_email": "admin@company.com",
  "created_at": "2020-01-15T10:00:00Z",
  "options": {
    "share_rc": true,
    "room_connectors": "zoomcrc.zoom.us",
    "pay_mode": "master",
    "meeting_capacity": 500
  },
  "vanity_url": "https://company.zoom.us"
}
```

## Example: Create Sub-Account

```bash
curl -X POST "https://api.zoom.us/v2/accounts" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "John",
    "last_name": "Doe",
    "email": "john.doe@subclient.com",
    "password": "SecureP@ss123!",
    "options": {
      "share_rc": true,
      "meeting_connectors": "zoomcrc.zoom.us",
      "pay_mode": "master"
    },
    "account_name": "SubClient Corp"
  }'
```

### Response

```json
{
  "id": "sub_abc123",
  "owner_id": "user_john",
  "owner_email": "john.doe@subclient.com",
  "created_at": "2024-01-25T10:00:00Z"
}
```

## Account Settings

### Get Settings

```bash
curl "https://api.zoom.us/v2/accounts/{accountId}/settings" \
  -H "Authorization: Bearer {accessToken}"
```

### Key Settings Categories

```json
{
  "schedule_meeting": {
    "host_video": true,
    "participants_video": false,
    "audio_type": "both",
    "join_before_host": false,
    "require_password_for_all_meetings": true
  },
  "in_meeting": {
    "chat": true,
    "allow_participants_chat_with": "everyone",
    "screen_sharing": true,
    "file_transfer": true,
    "virtual_background": true,
    "waiting_room": true
  },
  "email_notification": {
    "cloud_recording_available_reminder": true,
    "jbh_reminder": true,
    "cancel_meeting_reminder": true
  },
  "security": {
    "admin_change_name_pic": true,
    "import_photos_from_devices": true,
    "sign_in_with_two_factor_auth": "all"
  },
  "recording": {
    "local_recording": true,
    "cloud_recording": true,
    "auto_recording": "local",
    "auto_delete_cmr": true,
    "auto_delete_cmr_days": 30
  },
  "integration": {
    "google_calendar": true,
    "outlook_calendar": true,
    "slack": true
  }
}
```

### Update Settings

```bash
curl -X PATCH "https://api.zoom.us/v2/accounts/{accountId}/settings" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "schedule_meeting": {
      "require_password_for_all_meetings": true
    },
    "in_meeting": {
      "waiting_room": true
    },
    "security": {
      "sign_in_with_two_factor_auth": "all"
    }
  }'
```

## Lock Settings

Lock settings prevent sub-accounts from changing specific configurations:

```bash
curl -X PATCH "https://api.zoom.us/v2/accounts/{accountId}/lock_settings" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "schedule_meeting": {
      "require_password_for_all_meetings": true
    },
    "in_meeting": {
      "waiting_room": true
    }
  }'
```

## Plan Management

### Get Current Plans

```bash
curl "https://api.zoom.us/v2/accounts/{accountId}/plans" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "plan_base": {
    "type": "business",
    "hosts": 50
  },
  "plan_zoom_rooms": {
    "type": "zroom_yearly",
    "hosts": 10
  },
  "plan_webinar": [
    {
      "type": "webinar500_yearly",
      "hosts": 5
    }
  ],
  "plan_recording": "cmr_monthly_commitment_500",
  "plan_audio": {
    "type": "tollfree_monthly_commitment_100",
    "callout_countries": "US,CA,GB"
  }
}
```

## Trusted Domains

Manage email domains for SSO and security:

```bash
curl -X PATCH "https://api.zoom.us/v2/accounts/{accountId}/trusted_domains" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "trusted_domains": ["company.com", "subsidiary.com"]
  }'
```

## Account Types

| Type | Description |
|------|-------------|
| `master` | Parent account (ISV/reseller) |
| `sub` | Sub-account under master |
| `regular` | Standard account |

## Pay Modes

| Mode | Description |
|------|-------------|
| `master` | Master pays for sub-accounts |
| `sub` | Sub-accounts pay themselves |

## Webhooks

| Event | Description |
|-------|-------------|
| `account.created` | Sub-account created |
| `account.disassociated` | Sub-account removed |
| `account.settings_updated` | Settings changed |
| `account.upgraded` | Plan upgraded |

## Best Practices

1. **Use Master Account carefully** - Plan your account hierarchy
2. **Lock critical settings** - Enforce security policies
3. **Monitor sub-accounts** - Track usage and compliance
4. **Plan migrations** - Use API for bulk changes
5. **Backup settings** - Export before major changes

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Accounts
- **Account Management**: https://support.zoom.us/hc/en-us/categories/201146643-Account-Profile
- **Master Account Guide**: https://developers.zoom.us/docs/api/rest/accounts-guide/
