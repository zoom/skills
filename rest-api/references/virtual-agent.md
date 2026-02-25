# Zoom Virtual Agent API

AI-powered virtual agent for automated customer service.

## Overview

Zoom Virtual Agent (ZVA) is an AI-driven conversational chatbot that handles customer inquiries using natural language processing. The API enables management of virtual agents, conversation flows, intents, and knowledge bases.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with virtual agent scopes:
- `virtual_agent:read` - Read virtual agent data
- `virtual_agent:write` - Manage virtual agents
- `virtual_agent:read:admin` - Admin read access
- `virtual_agent:write:admin` - Admin write access

## Key Endpoint Categories

### Virtual Agents

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/virtual_agents` | List virtual agents |
| POST | `/virtual_agents` | Create virtual agent |
| GET | `/virtual_agents/{agentId}` | Get agent details |
| PATCH | `/virtual_agents/{agentId}` | Update agent |
| DELETE | `/virtual_agents/{agentId}` | Delete agent |
| POST | `/virtual_agents/{agentId}/publish` | Publish agent |

### Intents

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/virtual_agents/{agentId}/intents` | List intents |
| POST | `/virtual_agents/{agentId}/intents` | Create intent |
| GET | `/virtual_agents/{agentId}/intents/{intentId}` | Get intent details |
| PATCH | `/virtual_agents/{agentId}/intents/{intentId}` | Update intent |
| DELETE | `/virtual_agents/{agentId}/intents/{intentId}` | Delete intent |

### Entities

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/virtual_agents/{agentId}/entities` | List entities |
| POST | `/virtual_agents/{agentId}/entities` | Create entity |
| GET | `/virtual_agents/{agentId}/entities/{entityId}` | Get entity details |
| PATCH | `/virtual_agents/{agentId}/entities/{entityId}` | Update entity |
| DELETE | `/virtual_agents/{agentId}/entities/{entityId}` | Delete entity |

### Knowledge Base

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/virtual_agents/{agentId}/knowledge_base` | Get knowledge base |
| POST | `/virtual_agents/{agentId}/knowledge_base/articles` | Add article |
| PATCH | `/virtual_agents/{agentId}/knowledge_base/articles/{articleId}` | Update article |
| DELETE | `/virtual_agents/{agentId}/knowledge_base/articles/{articleId}` | Delete article |
| POST | `/virtual_agents/{agentId}/knowledge_base/sync` | Sync external KB |

### Conversations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/virtual_agents/{agentId}/conversations` | List conversations |
| GET | `/virtual_agents/{agentId}/conversations/{conversationId}` | Get conversation |
| GET | `/virtual_agents/{agentId}/conversations/{conversationId}/transcript` | Get transcript |

### Flows

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/virtual_agents/{agentId}/flows` | List conversation flows |
| POST | `/virtual_agents/{agentId}/flows` | Create flow |
| GET | `/virtual_agents/{agentId}/flows/{flowId}` | Get flow details |
| PATCH | `/virtual_agents/{agentId}/flows/{flowId}` | Update flow |

### Analytics

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/virtual_agents/{agentId}/analytics` | Get agent analytics |
| GET | `/virtual_agents/{agentId}/analytics/intents` | Intent analytics |
| GET | `/virtual_agents/{agentId}/analytics/resolution` | Resolution rates |

## Example: Create an Intent

```bash
curl -X POST "https://api.zoom.us/v2/virtual_agents/{agentId}/intents" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "check_order_status",
    "display_name": "Check Order Status",
    "training_phrases": [
      "Where is my order?",
      "Track my package",
      "What is my order status?",
      "When will my order arrive?"
    ],
    "responses": [
      {
        "type": "text",
        "text": "I can help you check your order status. Could you please provide your order number?"
      }
    ]
  }'
```

### Response

```json
{
  "id": "intent_abc123",
  "name": "check_order_status",
  "display_name": "Check Order Status",
  "training_phrases_count": 4,
  "created_at": "2024-01-15T10:00:00Z"
}
```

## Example: Add Knowledge Base Article

```bash
curl -X POST "https://api.zoom.us/v2/virtual_agents/{agentId}/knowledge_base/articles" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Return Policy",
    "content": "Our return policy allows returns within 30 days of purchase...",
    "category": "Policies",
    "keywords": ["return", "refund", "exchange"],
    "url": "https://example.com/help/returns"
  }'
```

## Example: Get Agent Analytics

```bash
curl -X GET "https://api.zoom.us/v2/virtual_agents/{agentId}/analytics?from=2024-01-01&to=2024-01-31" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "period": {
    "from": "2024-01-01",
    "to": "2024-01-31"
  },
  "total_conversations": 5420,
  "resolved_by_agent": 4200,
  "escalated_to_human": 1220,
  "resolution_rate": 77.5,
  "avg_conversation_duration": 180,
  "top_intents": [
    {
      "intent": "check_order_status",
      "count": 1850
    },
    {
      "intent": "return_request",
      "count": 980
    }
  ]
}
```

## Response Types

| Type | Description |
|------|-------------|
| `text` | Plain text response |
| `card` | Rich card with buttons |
| `carousel` | Multiple cards |
| `quick_replies` | Suggested responses |
| `handoff` | Escalate to human agent |
| `api_call` | Call external API |

## Entity Types

| Type | Description |
|------|-------------|
| `system` | Built-in entities (date, number, etc.) |
| `custom` | Custom defined entities |
| `regex` | Pattern-based entities |
| `list` | List of values |

## Conversation Status

| Status | Description |
|--------|-------------|
| `active` | Conversation in progress |
| `resolved` | Resolved by virtual agent |
| `escalated` | Handed off to human |
| `abandoned` | Customer left |

## Channel Integration

Virtual Agent can be deployed across:
- Web chat widget
- Mobile app
- Zoom Contact Center
- Messaging platforms

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Virtual-Agent
- **Virtual Agent Documentation**: https://developers.zoom.us/docs/virtual-agent/
- **Bot Building Guide**: https://developers.zoom.us/docs/virtual-agent/building/
