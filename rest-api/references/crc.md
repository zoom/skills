# Zoom CRC (Cloud Room Connector) API

Connect SIP/H.323 conference room systems to Zoom meetings.

## Overview

Cloud Room Connector (CRC) enables traditional video conferencing systems (Cisco, Polycom, etc.) to join Zoom meetings via SIP or H.323 protocols. The API manages CRC configurations, dial-in settings, and room system integrations.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with CRC scopes:
- `crc:read` - Read CRC configurations
- `crc:write` - Manage CRC settings
- `crc:read:admin` - Admin read access
- `crc:write:admin` - Admin write access

## Key Endpoints

### CRC Settings

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/accounts/{accountId}/crc` | Get CRC settings |
| PATCH | `/accounts/{accountId}/crc` | Update CRC settings |
| GET | `/accounts/{accountId}/crc/ports` | Get CRC port usage |

### SIP Devices

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/sip_phones` | List SIP devices |
| POST | `/sip_phones` | Register SIP device |
| GET | `/sip_phones/{deviceId}` | Get device details |
| PATCH | `/sip_phones/{deviceId}` | Update device |
| DELETE | `/sip_phones/{deviceId}` | Remove device |

### H.323 Devices

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/h323/devices` | List H.323 devices |
| POST | `/h323/devices` | Register H.323 device |
| GET | `/h323/devices/{deviceId}` | Get device details |
| PATCH | `/h323/devices/{deviceId}` | Update device |
| DELETE | `/h323/devices/{deviceId}` | Remove device |

### Room Connectors

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rooms/connectors` | List room connectors |
| GET | `/rooms/connectors/{connectorId}` | Get connector details |

### Dial-In Information

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/meetings/{meetingId}/sip_dialing` | Get SIP dial-in info |
| GET | `/meetings/{meetingId}/h323_dialing` | Get H.323 dial-in info |

## Example: Get CRC Settings

```bash
curl -X GET "https://api.zoom.us/v2/accounts/{accountId}/crc" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "enabled": true,
  "sip_uri": "meeting_id@zoomcrc.com",
  "h323_ip_addresses": [
    "162.255.37.11",
    "162.255.36.11"
  ],
  "ports_allocated": 10,
  "ports_in_use": 3
}
```

## Example: Register SIP Device

```bash
curl -X POST "https://api.zoom.us/v2/sip_phones" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "authorization_name": "conf-room-1",
    "domain": "sip.company.com",
    "user_name": "conference_room_1",
    "password": "secure_password",
    "voice_mail": "Conference Room 1",
    "registration_expire_time": 60
  }'
```

## Example: Get Meeting SIP Dial-In

```bash
curl -X GET "https://api.zoom.us/v2/meetings/{meetingId}/sip_dialing" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "sip_dialing": {
    "uri": "12345678910@zoomcrc.com",
    "password": "abc123",
    "ip_addresses": [
      "162.255.37.11",
      "162.255.36.11"
    ]
  }
}
```

## SIP/H.323 Dial Strings

### SIP URI Format
```
{meeting_id}.{passcode}@zoomcrc.com
```

### H.323 Format
```
IP: 162.255.37.11
Meeting ID: {meeting_id}
Passcode: {passcode}
```

## Device Registration Fields

| Field | Description |
|-------|-------------|
| `authorization_name` | Unique device identifier |
| `domain` | SIP domain |
| `user_name` | Registration username |
| `password` | Registration password |
| `transport_protocol` | UDP, TCP, TLS, AUTO |
| `registration_expire_time` | Expiration in seconds |

## Port Management

| Field | Description |
|-------|-------------|
| `ports_allocated` | Total CRC ports for account |
| `ports_in_use` | Currently active connections |
| `ports_available` | Remaining capacity |

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Cloud-Room-Connector
- **CRC Documentation**: https://developers.zoom.us/docs/crc/
- **SIP/H.323 Room Connector Guide**: https://support.zoom.us/hc/en-us/articles/201363273
