# Zoom Revenue Accelerator API

Conversation intelligence and sales analytics API.

## Overview

Zoom Revenue Accelerator (ZRA) provides conversation intelligence for sales teams, analyzing calls and meetings to provide insights, coaching opportunities, and deal intelligence. The API enables access to conversation analytics, deal tracking, and coaching metrics.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with revenue accelerator scopes:
- `revenue_accelerator:read` - Read analytics data
- `revenue_accelerator:write` - Manage settings
- `revenue_accelerator:read:admin` - Admin read access
- `revenue_accelerator:write:admin` - Admin write access

## Key Endpoint Categories

### Conversations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/revenue_accelerator/conversations` | List conversations |
| GET | `/revenue_accelerator/conversations/{conversationId}` | Get conversation details |
| GET | `/revenue_accelerator/conversations/{conversationId}/transcript` | Get transcript |
| GET | `/revenue_accelerator/conversations/{conversationId}/insights` | Get AI insights |

### Deals

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/revenue_accelerator/deals` | List deals |
| GET | `/revenue_accelerator/deals/{dealId}` | Get deal details |
| GET | `/revenue_accelerator/deals/{dealId}/conversations` | Get deal conversations |
| GET | `/revenue_accelerator/deals/{dealId}/health` | Get deal health score |

### Analytics

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/revenue_accelerator/analytics/team` | Team analytics |
| GET | `/revenue_accelerator/analytics/rep/{userId}` | Rep analytics |
| GET | `/revenue_accelerator/analytics/trends` | Trend analysis |
| GET | `/revenue_accelerator/analytics/pipeline` | Pipeline analytics |

### Coaching

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/revenue_accelerator/coaching/scorecards` | List scorecards |
| GET | `/revenue_accelerator/coaching/scorecards/{scorecardId}` | Get scorecard |
| GET | `/revenue_accelerator/coaching/rep/{userId}` | Rep coaching metrics |
| GET | `/revenue_accelerator/coaching/moments` | Coaching moments |

### Topics & Keywords

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/revenue_accelerator/topics` | List tracked topics |
| POST | `/revenue_accelerator/topics` | Create topic tracker |
| GET | `/revenue_accelerator/topics/{topicId}/mentions` | Get topic mentions |
| GET | `/revenue_accelerator/keywords` | List tracked keywords |

### Alerts

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/revenue_accelerator/alerts` | List alerts |
| POST | `/revenue_accelerator/alerts` | Create alert rule |
| PATCH | `/revenue_accelerator/alerts/{alertId}` | Update alert |

## Example: List Conversations

```bash
curl -X GET "https://api.zoom.us/v2/revenue_accelerator/conversations?from=2024-01-01&to=2024-01-31&page_size=50" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "page_size": 50,
  "next_page_token": "abc123",
  "conversations": [
    {
      "id": "conv_xyz",
      "title": "Discovery Call - Acme Corp",
      "date": "2024-01-15T14:00:00Z",
      "duration": 1800,
      "participants": [
        {"name": "John Sales", "role": "rep"},
        {"name": "Jane Buyer", "role": "prospect"}
      ],
      "deal": {
        "id": "deal_abc",
        "name": "Acme Corp - Enterprise"
      },
      "insights": {
        "talk_ratio": 0.45,
        "questions_asked": 12,
        "topics_mentioned": ["pricing", "implementation"],
        "sentiment": "positive"
      }
    }
  ]
}
```

## Example: Get Conversation Insights

```bash
curl -X GET "https://api.zoom.us/v2/revenue_accelerator/conversations/{conversationId}/insights" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "conversation_id": "conv_xyz",
  "insights": {
    "talk_ratio": {
      "rep": 0.45,
      "prospect": 0.55
    },
    "talk_speed": {
      "rep_wpm": 145,
      "prospect_wpm": 138
    },
    "engagement": {
      "score": 82,
      "questions_asked": 12,
      "questions_answered": 8,
      "longest_monologue_seconds": 90
    },
    "topics": [
      {"name": "pricing", "duration_seconds": 180, "sentiment": "neutral"},
      {"name": "competitors", "duration_seconds": 120, "sentiment": "positive"},
      {"name": "timeline", "duration_seconds": 90, "sentiment": "positive"}
    ],
    "action_items": [
      "Send pricing proposal by Friday",
      "Schedule technical demo with IT team"
    ],
    "next_steps_discussed": true,
    "competitor_mentions": ["CompetitorA", "CompetitorB"]
  }
}
```

## Example: Get Deal Health

```bash
curl -X GET "https://api.zoom.us/v2/revenue_accelerator/deals/{dealId}/health" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "deal_id": "deal_abc",
  "deal_name": "Acme Corp - Enterprise",
  "health_score": 78,
  "factors": {
    "engagement": {
      "score": 85,
      "description": "High engagement with multiple stakeholders"
    },
    "momentum": {
      "score": 70,
      "description": "Steady progress, slight slowdown this week"
    },
    "relationship": {
      "score": 80,
      "description": "Strong relationship with champion"
    },
    "competition": {
      "score": 75,
      "description": "Moderate competitive pressure"
    }
  },
  "risks": [
    "No executive sponsor identified",
    "Budget discussion not yet complete"
  ],
  "recommendations": [
    "Schedule call with economic buyer",
    "Address budget concerns proactively"
  ]
}
```

## Insight Metrics

| Metric | Description |
|--------|-------------|
| `talk_ratio` | Speaking time distribution |
| `talk_speed` | Words per minute |
| `questions_asked` | Number of questions |
| `filler_words` | Filler word usage |
| `sentiment` | Conversation sentiment |
| `engagement_score` | Overall engagement |

## Topic Categories

| Category | Description |
|----------|-------------|
| `pricing` | Pricing discussions |
| `competitors` | Competitor mentions |
| `objections` | Objection handling |
| `next_steps` | Next step discussions |
| `timeline` | Timeline/urgency |
| `budget` | Budget discussions |

## Alert Types

| Type | Description |
|------|-------------|
| `competitor_mention` | Competitor mentioned |
| `objection_raised` | Objection detected |
| `deal_risk` | Deal health declining |
| `coaching_opportunity` | Coaching moment identified |

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Revenue-Accelerator
- **Revenue Accelerator Documentation**: https://developers.zoom.us/docs/revenue-accelerator/
- **Conversation Intelligence Guide**: https://developers.zoom.us/docs/revenue-accelerator/insights/
