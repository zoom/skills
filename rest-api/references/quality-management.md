# Quality Management API

Call quality management and analytics for Zoom Contact Center.

## Overview

Quality Management APIs provide tools for monitoring, evaluating, and improving agent performance in contact center operations. These endpoints enable supervisors to score interactions, track quality metrics, and manage evaluation workflows.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with Contact Center scopes:
- `contact_center:read:quality_management` - Read quality data
- `contact_center:write:quality_management` - Create/modify evaluations
- `contact_center:read:quality_management:admin` - Admin read access

## Key Endpoints

### Evaluations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/quality_management/evaluations` | List evaluations |
| POST | `/contact_center/quality_management/evaluations` | Create evaluation |
| GET | `/contact_center/quality_management/evaluations/{evaluationId}` | Get evaluation details |
| PATCH | `/contact_center/quality_management/evaluations/{evaluationId}` | Update evaluation |
| DELETE | `/contact_center/quality_management/evaluations/{evaluationId}` | Delete evaluation |

### Scorecards

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/quality_management/scorecards` | List scorecards |
| POST | `/contact_center/quality_management/scorecards` | Create scorecard |
| GET | `/contact_center/quality_management/scorecards/{scorecardId}` | Get scorecard |
| PATCH | `/contact_center/quality_management/scorecards/{scorecardId}` | Update scorecard |

### Agent Metrics

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/quality_management/agents/{agentId}/metrics` | Get agent quality metrics |
| GET | `/contact_center/quality_management/agents/{agentId}/evaluations` | List agent evaluations |
| GET | `/contact_center/quality_management/teams/{teamId}/metrics` | Get team metrics |

### Calibration

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/contact_center/quality_management/calibrations` | List calibration sessions |
| POST | `/contact_center/quality_management/calibrations` | Create calibration session |

## Example: Create Evaluation

```bash
curl -X POST "https://api.zoom.us/v2/contact_center/quality_management/evaluations" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "engagement_id": "eng_abc123",
    "agent_id": "agent_xyz",
    "scorecard_id": "sc_123",
    "scores": [
      {
        "criterion_id": "crit_1",
        "score": 4,
        "comment": "Excellent greeting"
      },
      {
        "criterion_id": "crit_2", 
        "score": 3,
        "comment": "Good problem resolution"
      }
    ],
    "overall_comment": "Strong performance overall"
  }'
```

## Scorecard Structure

```json
{
  "id": "sc_123",
  "name": "Customer Service Scorecard",
  "categories": [
    {
      "name": "Opening",
      "weight": 20,
      "criteria": [
        {
          "id": "crit_1",
          "name": "Professional Greeting",
          "max_score": 5
        }
      ]
    },
    {
      "name": "Resolution",
      "weight": 50,
      "criteria": [
        {
          "id": "crit_2",
          "name": "Problem Understanding",
          "max_score": 5
        },
        {
          "id": "crit_3",
          "name": "Solution Accuracy",
          "max_score": 5
        }
      ]
    }
  ]
}
```

## Quality Metrics

| Metric | Description |
|--------|-------------|
| `average_score` | Average evaluation score |
| `evaluation_count` | Number of evaluations |
| `trend` | Score trend over time |
| `percentile_rank` | Agent ranking |
| `compliance_rate` | Compliance with required behaviors |

## Use Cases

- Agent performance tracking
- Quality compliance monitoring
- Training needs identification
- Calibration and consistency
- Trend analysis and reporting

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/contact-center/methods/
- **Contact Center Documentation**: https://developers.zoom.us/docs/contact-center/
