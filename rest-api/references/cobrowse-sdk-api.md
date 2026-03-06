# Co-browse SDK API

Server-side APIs for Zoom Co-browse session management.

## Overview

The Co-browse SDK API provides endpoints for managing co-browsing sessions, allowing support agents to view and interact with customer screens in real-time. These APIs enable session initiation, control, and analytics for co-browse implementations.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with co-browse scopes:
- `cobrowse:read` - Read session data
- `cobrowse:write` - Create/manage sessions
- `cobrowse:read:admin` - Admin access

## Key Endpoints

### Sessions

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/cobrowse/sessions` | Create co-browse session |
| GET | `/cobrowse/sessions/{sessionId}` | Get session details |
| DELETE | `/cobrowse/sessions/{sessionId}` | End session |
| GET | `/cobrowse/sessions` | List sessions |

### Session Control

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/cobrowse/sessions/{sessionId}/request-control` | Request control |
| POST | `/cobrowse/sessions/{sessionId}/release-control` | Release control |
| POST | `/cobrowse/sessions/{sessionId}/highlight` | Send highlight |
| POST | `/cobrowse/sessions/{sessionId}/annotation` | Add annotation |

### Session Participants

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/cobrowse/sessions/{sessionId}/participants` | List participants |
| POST | `/cobrowse/sessions/{sessionId}/invite` | Invite participant |
| DELETE | `/cobrowse/sessions/{sessionId}/participants/{userId}` | Remove participant |

### Reports

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/cobrowse/reports/sessions` | Session usage report |
| GET | `/cobrowse/reports/sessions/{sessionId}` | Session detail report |

## Example: Create Co-browse Session

```bash
curl -X POST "https://api.zoom.us/v2/cobrowse/sessions" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "customer_id": "cust_123",
    "agent_id": "agent_456",
    "session_type": "screen_share",
    "settings": {
      "control_enabled": true,
      "annotation_enabled": true,
      "highlight_enabled": true,
      "masking_enabled": true,
      "recording_enabled": false
    },
    "context": {
      "case_id": "case_789",
      "department": "support"
    }
  }'
```

### Response

```json
{
  "id": "cob_abc123",
  "customer_id": "cust_123",
  "agent_id": "agent_456",
  "status": "pending",
  "created_at": "2024-01-25T10:00:00Z",
  "customer_join_url": "https://zoom.us/cobrowse/join/abc123?token=xxx",
  "agent_join_url": "https://zoom.us/cobrowse/agent/abc123?token=yyy",
  "session_code": "ABC-123-XYZ"
}
```

## Example: Get Session Details

```bash
curl "https://api.zoom.us/v2/cobrowse/sessions/{sessionId}" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "id": "cob_abc123",
  "customer_id": "cust_123",
  "agent_id": "agent_456",
  "status": "active",
  "session_type": "screen_share",
  "started_at": "2024-01-25T10:05:00Z",
  "duration_seconds": 300,
  "participants": [
    {
      "id": "cust_123",
      "role": "customer",
      "name": "John Customer",
      "joined_at": "2024-01-25T10:05:00Z"
    },
    {
      "id": "agent_456",
      "role": "agent",
      "name": "Jane Agent",
      "joined_at": "2024-01-25T10:05:30Z",
      "has_control": false
    }
  ],
  "settings": {
    "control_enabled": true,
    "annotation_enabled": true
  }
}
```

## Example: Request Control

```bash
curl -X POST "https://api.zoom.us/v2/cobrowse/sessions/{sessionId}/request-control" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "agent_456",
    "control_type": "full",
    "message": "May I take control to help fill out the form?"
  }'
```

### Response

```json
{
  "request_id": "req_xyz",
  "status": "pending_approval",
  "requested_at": "2024-01-25T10:10:00Z"
}
```

## Example: Send Highlight

```bash
curl -X POST "https://api.zoom.us/v2/cobrowse/sessions/{sessionId}/highlight" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "element_selector": "#submit-button",
    "highlight_type": "pulse",
    "duration_seconds": 5,
    "message": "Click this button to continue"
  }'
```

## Session Types

| Type | Description |
|------|-------------|
| `screen_share` | Customer shares entire screen |
| `app_share` | Customer shares specific application |
| `browser_share` | Customer shares browser tab |
| `embedded` | Co-browse within embedded widget |

## Session Status

| Status | Description |
|--------|-------------|
| `pending` | Session created, awaiting join |
| `active` | Session in progress |
| `paused` | Session temporarily paused |
| `ended` | Session completed |
| `expired` | Session timed out |

## Control Types

| Type | Description |
|------|-------------|
| `full` | Full mouse and keyboard control |
| `limited` | Click and scroll only |
| `view_only` | No control, view only |

## Data Masking

Configure sensitive data masking:

```json
{
  "masking": {
    "enabled": true,
    "rules": [
      {
        "type": "css_selector",
        "selector": "input[type='password']"
      },
      {
        "type": "css_selector",
        "selector": ".credit-card-number"
      },
      {
        "type": "pattern",
        "regex": "\\d{4}-\\d{4}-\\d{4}-\\d{4}"
      }
    ]
  }
}
```

## Client Integration

### Customer Side (JavaScript)

```javascript
import { CobrowseSDK } from '@zoom/zoom-cobrowse-sdk';

const cobrowse = new CobrowseSDK({
  apiKey: 'your_api_key'
});

// Start session with code
await cobrowse.joinSession('ABC-123-XYZ');

// Handle control requests
cobrowse.on('control_requested', (request) => {
  if (confirm(request.message)) {
    cobrowse.grantControl(request.request_id);
  } else {
    cobrowse.denyControl(request.request_id);
  }
});

// Handle highlights
cobrowse.on('highlight', (highlight) => {
  // SDK automatically shows highlight
  console.log('Highlight message:', highlight.message);
});
```

### Agent Side

```javascript
const cobrowse = new CobrowseAgentSDK({
  apiKey: 'your_api_key'
});

await cobrowse.joinAsAgent(sessionId);

// Request control
await cobrowse.requestControl({
  controlType: 'full',
  message: 'May I help fill this form?'
});

// Send highlight
await cobrowse.highlight({
  selector: '#next-button',
  type: 'pulse',
  message: 'Click here to proceed'
});

// Add annotation
await cobrowse.annotate({
  type: 'arrow',
  x: 500,
  y: 300,
  message: 'Start here'
});
```

## Webhooks

| Event | Description |
|-------|-------------|
| `cobrowse.session.created` | Session created |
| `cobrowse.session.started` | Session began |
| `cobrowse.session.ended` | Session ended |
| `cobrowse.control.requested` | Control requested |
| `cobrowse.control.granted` | Control granted |
| `cobrowse.control.released` | Control released |

## Security Considerations

1. **Always use data masking** - Hide sensitive fields
2. **Require explicit consent** - Customer must approve sharing
3. **Control approval** - Customer approves each control request
4. **Audit logging** - All actions are logged
5. **Session timeouts** - Auto-end inactive sessions

## Best Practices

1. **Clear communication** - Explain actions to customer
2. **Minimal control time** - Release control when done
3. **Use highlights** - Guide without taking control
4. **Mask sensitive data** - Configure masking rules
5. **Test thoroughly** - Verify masking works

## Resources

- **Co-browse SDK Guide**: https://developers.zoom.us/docs/cobrowse-sdk/
- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/cobrowse-api/methods/
- **Security Best Practices**: https://developers.zoom.us/docs/cobrowse-sdk/security/
