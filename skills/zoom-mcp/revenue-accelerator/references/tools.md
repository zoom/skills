# Revenue Accelerator MCP Tools

| Tool | Scope |
|------|-------|
| `search_conversations` | `zra:read:list_conversations` |
| `get_conversation_transcript` | `zra:read:conversations` |
| `get_conversation_analysis` | `zra:read:conversation_analysis` |
| `get_conversation_comments` | `zra:read:list_conversation_comments` |
| `get_scorecard_sessions` | `zra:read:conversation_scorecards` |
| `search_deals` | `zra:read:list_deals` |
| `get_deal_detail_v2` | `zra:read:deal` |
| `get_deal_analysis` | `zra:read:deal` |
| `get_deal_stages` | `zra:read:deal` |
| `get_deal_activities_v2` | `zra:read:list_deal_activities` |
| `get_customer_accounts` | `zra:read:crm_customer_contact` |
| `get_customer_contacts` | `zra:read:crm_customer_contact` |
| `search_indicators` | `zra:read:indicator` |
| `get_manager_team_and_member` | `zra:read:team` |
| `search_internal_users` | `zra:read:user` |

Protected-resource metadata can also advertise legacy alias scopes. Prefer the granular `zra:*`
scope documented beside each current tool, then verify the minted token and `tools/list` at runtime.
