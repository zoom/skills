# Healthcare API

HIPAA-compliant telehealth and healthcare integration endpoints.

## Overview

Zoom's Healthcare APIs provide HIPAA-compliant endpoints for telehealth applications, patient engagement, and healthcare workflow integration. These APIs are available to customers with Zoom for Healthcare licenses.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with healthcare-specific scopes:
- `healthcare:read` - Read healthcare data
- `healthcare:write` - Create/modify healthcare records
- Healthcare BAA (Business Associate Agreement) required

## Important: HIPAA Compliance

To use Healthcare APIs:
1. Must have Zoom for Healthcare license
2. Must sign BAA with Zoom
3. Must enable HIPAA-compliant settings
4. Must use approved authentication methods

## Key Endpoints

### Virtual Visits

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/healthcare/visits` | Schedule virtual visit |
| GET | `/healthcare/visits/{visitId}` | Get visit details |
| PATCH | `/healthcare/visits/{visitId}` | Update visit |
| DELETE | `/healthcare/visits/{visitId}` | Cancel visit |
| GET | `/healthcare/visits` | List visits |

### Patient Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/healthcare/patients` | Create patient record |
| GET | `/healthcare/patients/{patientId}` | Get patient info |
| PATCH | `/healthcare/patients/{patientId}` | Update patient |
| POST | `/healthcare/patients/{patientId}/invite` | Send visit invitation |

### Waiting Room

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/healthcare/waiting_room` | Get waiting room status |
| POST | `/healthcare/waiting_room/{patientId}/admit` | Admit patient |
| POST | `/healthcare/waiting_room/{patientId}/notify` | Send notification |

### Provider Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/healthcare/providers` | List providers |
| GET | `/healthcare/providers/{providerId}/availability` | Get availability |
| PATCH | `/healthcare/providers/{providerId}/availability` | Set availability |

## Example: Schedule Virtual Visit

```bash
curl -X POST "https://api.zoom.us/v2/healthcare/visits" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "patient_id": "pat_123",
    "provider_id": "prov_456",
    "scheduled_time": "2024-02-01T14:00:00Z",
    "duration_minutes": 30,
    "visit_type": "follow_up",
    "reason": "Post-surgery checkup",
    "settings": {
      "waiting_room": true,
      "recording": false,
      "require_encryption": true
    },
    "notifications": {
      "patient_email": true,
      "patient_sms": true,
      "reminder_minutes": [1440, 60]
    }
  }'
```

### Response

```json
{
  "id": "visit_abc123",
  "patient_id": "pat_123",
  "provider_id": "prov_456",
  "scheduled_time": "2024-02-01T14:00:00Z",
  "duration_minutes": 30,
  "status": "scheduled",
  "join_url_patient": "https://zoom.us/j/123?pwd=xxx",
  "join_url_provider": "https://zoom.us/j/123?pwd=yyy",
  "created_at": "2024-01-25T10:00:00Z"
}
```

## Example: Send Patient Invitation

```bash
curl -X POST "https://api.zoom.us/v2/healthcare/patients/{patientId}/invite" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "visit_id": "visit_abc123",
    "channels": ["email", "sms"],
    "message": "Your telehealth appointment is scheduled for Feb 1 at 2:00 PM."
  }'
```

## Visit Types

| Type | Description |
|------|-------------|
| `initial_consultation` | First visit with provider |
| `follow_up` | Follow-up appointment |
| `urgent_care` | Urgent/same-day visit |
| `mental_health` | Behavioral health session |
| `group_therapy` | Group session |
| `specialist_consult` | Specialist referral |

## Visit Status

| Status | Description |
|--------|-------------|
| `scheduled` | Appointment scheduled |
| `waiting` | Patient in waiting room |
| `in_progress` | Visit ongoing |
| `completed` | Visit finished |
| `cancelled` | Visit cancelled |
| `no_show` | Patient didn't attend |

## HIPAA-Compliant Settings

Required settings for HIPAA compliance:

```json
{
  "meeting_settings": {
    "waiting_room": true,
    "password_required": true,
    "encryption_type": "e2ee",
    "allow_recording": false,
    "authenticated_users_only": true,
    "watermark": true
  },
  "data_retention": {
    "auto_delete_days": 30,
    "exclude_from_analytics": true
  }
}
```

## EHR Integration

### Supported EHR Systems

- Epic (via FHIR)
- Cerner (via FHIR)
- Allscripts
- athenahealth
- Custom integrations via API

### FHIR Resources

```json
{
  "resourceType": "Appointment",
  "status": "booked",
  "participant": [
    {
      "actor": {
        "reference": "Patient/pat_123"
      }
    },
    {
      "actor": {
        "reference": "Practitioner/prov_456"
      }
    }
  ],
  "extension": [
    {
      "url": "http://zoom.us/fhir/telehealth",
      "valueString": "visit_abc123"
    }
  ]
}
```

## Webhooks

Healthcare-specific webhook events:

| Event | Description |
|-------|-------------|
| `healthcare.visit.scheduled` | Visit created |
| `healthcare.visit.started` | Visit began |
| `healthcare.visit.ended` | Visit completed |
| `healthcare.visit.cancelled` | Visit cancelled |
| `healthcare.patient.waiting` | Patient in waiting room |
| `healthcare.patient.admitted` | Patient admitted |

## Security Requirements

| Requirement | Implementation |
|-------------|----------------|
| Encryption in transit | TLS 1.2+ required |
| Encryption at rest | AES-256 |
| Access logging | All API calls logged |
| Session timeout | 15 minutes inactivity |
| MFA | Required for providers |

## Best Practices

1. **Always use waiting rooms** - Prevents unauthorized access
2. **Enable end-to-end encryption** - Maximum privacy
3. **Disable cloud recording** - Unless explicit consent
4. **Use authenticated join** - Verify participants
5. **Implement audit logging** - Track all access
6. **Set data retention policies** - Automatic deletion

## Resources

- **Zoom for Healthcare**: https://zoom.us/healthcare
- **HIPAA Compliance**: https://zoom.us/docs/doc/Zoom-hipaa.pdf
- **BAA Information**: https://zoom.us/healthcare/baa
- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/
