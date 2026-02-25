# Workforce Management API

Agent scheduling, forecasting, and capacity planning for Zoom Contact Center.

## Overview

Workforce Management (WFM) APIs enable contact center managers to optimize staffing levels, create agent schedules, forecast demand, and track adherence. These tools help ensure the right number of agents are available at the right times.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with Contact Center scopes:
- `contact_center:read:workforce_management` - Read WFM data
- `contact_center:write:workforce_management` - Create/modify schedules
- `contact_center:read:workforce_management:admin` - Admin access

## Key Endpoints

### Schedules

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/workforce_management/schedules` | List schedules |
| POST | `/contact_center/workforce_management/schedules` | Create schedule |
| GET | `/contact_center/workforce_management/schedules/{scheduleId}` | Get schedule |
| PATCH | `/contact_center/workforce_management/schedules/{scheduleId}` | Update schedule |
| DELETE | `/contact_center/workforce_management/schedules/{scheduleId}` | Delete schedule |

### Shifts

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/workforce_management/shifts` | List shifts |
| POST | `/contact_center/workforce_management/shifts` | Create shift |
| GET | `/contact_center/workforce_management/agents/{agentId}/shifts` | Get agent shifts |
| POST | `/contact_center/workforce_management/shifts/swap` | Request shift swap |

### Forecasting

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/workforce_management/forecasts` | List forecasts |
| POST | `/contact_center/workforce_management/forecasts` | Generate forecast |
| GET | `/contact_center/workforce_management/forecasts/{forecastId}` | Get forecast details |

### Adherence

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/workforce_management/adherence` | Get real-time adherence |
| GET | `/contact_center/workforce_management/adherence/historical` | Get historical adherence |
| GET | `/contact_center/workforce_management/agents/{agentId}/adherence` | Get agent adherence |

### Time Off

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/workforce_management/time_off` | List time off requests |
| POST | `/contact_center/workforce_management/time_off` | Request time off |
| PATCH | `/contact_center/workforce_management/time_off/{requestId}` | Approve/deny request |

## Example: Create Schedule

```bash
curl -X POST "https://api.zoom.us/v2/contact_center/workforce_management/schedules" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Week 5 Schedule",
    "start_date": "2024-02-01",
    "end_date": "2024-02-07",
    "team_id": "team_abc",
    "shifts": [
      {
        "agent_id": "agent_123",
        "date": "2024-02-01",
        "start_time": "09:00",
        "end_time": "17:00",
        "breaks": [
          {
            "start_time": "12:00",
            "duration_minutes": 60,
            "type": "lunch"
          }
        ]
      }
    ]
  }'
```

## Example: Get Real-time Adherence

```bash
curl "https://api.zoom.us/v2/contact_center/workforce_management/adherence?team_id=team_abc" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "agents": [
    {
      "agent_id": "agent_123",
      "name": "John Doe",
      "scheduled_state": "available",
      "actual_state": "available",
      "adherence_status": "in_adherence",
      "out_of_adherence_minutes": 0
    },
    {
      "agent_id": "agent_456",
      "name": "Jane Smith",
      "scheduled_state": "available",
      "actual_state": "break",
      "adherence_status": "out_of_adherence",
      "out_of_adherence_minutes": 5
    }
  ],
  "overall_adherence_percentage": 85.5
}
```

## Forecast Model

```json
{
  "id": "fc_123",
  "period": {
    "start": "2024-02-01",
    "end": "2024-02-28"
  },
  "intervals": [
    {
      "datetime": "2024-02-01T09:00:00Z",
      "forecasted_contacts": 45,
      "forecasted_handle_time": 300,
      "required_agents": 8,
      "confidence": 0.85
    }
  ],
  "methodology": "historical_pattern",
  "accuracy_metrics": {
    "mape": 12.5,
    "rmse": 8.3
  }
}
```

## Adherence States

| State | Description |
|-------|-------------|
| `in_adherence` | Agent state matches schedule |
| `out_of_adherence` | Agent state differs from schedule |
| `exception` | Approved deviation |
| `unknown` | Cannot determine adherence |

## Shift Types

| Type | Description |
|------|-------------|
| `regular` | Standard shift |
| `overtime` | Extra hours |
| `on_call` | Available if needed |
| `training` | Training session |
| `meeting` | Team meeting |

## Metrics

| Metric | Description |
|--------|-------------|
| `adherence_percentage` | % time in adherence |
| `conformance_percentage` | % of scheduled time worked |
| `occupancy` | % time handling contacts |
| `shrinkage` | Non-productive time |
| `service_level` | % calls answered within threshold |

## Use Cases

- Automated schedule generation
- Real-time adherence monitoring
- Demand forecasting
- Shift swap management
- Time off workflow automation

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/contact-center/methods/
- **Contact Center Documentation**: https://developers.zoom.us/docs/contact-center/
