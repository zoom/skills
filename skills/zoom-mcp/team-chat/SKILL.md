---
name: zoom-mcp/team-chat
description: |
  Zoom Team Chat MCP server guidance. Use for Team Chat MCP endpoints, OAuth scopes,
  and write-capable tool workflows such as sending or editing messages, creating or
  updating channels, adding channel members, and sending contact invitations. Prefer
  this child skill when the request is specifically about Team Chat MCP rather than
  default Zoom MCP search or deterministic Team Chat REST API implementation.
triggers:
  - "team chat mcp"
  - "zoom chat mcp"
  - "zoom team chat mcp"
  - "send zoom chat via mcp"
  - "edit zoom chat message mcp"
  - "create zoom chat channel mcp"
  - "add zoom chat channel members mcp"
  - "zoom_chat_message_send"
  - "zoom_chat_channel_create"
---

# Zoom MCP Team Chat

Dedicated guidance for Zoom's Team Chat MCP server.

Use this MCP surface for agent-driven Team Chat actions. Use
[../../team-chat/SKILL.md](../../team-chat/SKILL.md) when the user needs deterministic REST
API implementation, custom backend retries, webhooks, reporting, or a production integration
that should not depend on agent tool invocation.

## Endpoints

| Transport | URL |
|-----------|-----|
| Streamable HTTP (recommended) | `https://mcp.zoom.us/mcp/team_chat/streamable` |
| SSE (fallback) | `https://mcp.zoom.us/mcp/team_chat/sse` |

The repo MCP bundle registers this as `zoom-team-chat-mcp` in [../../../.mcp.json](../../../.mcp.json).

## Authentication

- OAuth bearer tokens are passed through the MCP `Authorization` header.
- The protected-resource metadata endpoint is `https://mcp.zoom.us/.well-known/oauth-protected-resource/mcp_team_chat`.
- The server is scoped to the caller's account and subject to Team Chat policy restrictions.
- End-to-end encrypted, archived, retention-restricted, or policy-blocked chats may not be accessible or mutable through this surface.

## Required Scopes

Team Chat MCP scopes advertised by protected-resource metadata:

- `team_chat:write:user_message`
- `team_chat:update:user_message`
- `team_chat:write:contact_information`
- `team_chat:write:user_channel`
- `team_chat:update:user_channel`
- `team_chat:write:members`

## Safety Rules

This server contains write-capable tools. Before invoking a write tool:

1. Confirm the user explicitly asked to send, edit, create, update, invite, or add members.
2. Resolve IDs carefully. `chat_session_id` may be a user/member ID, user email for 1:1 messages, or a channel ID.
3. Preview message/channel changes when the target or wording is ambiguous.
4. Do not guess channel IDs, message IDs, or invitee emails.
5. Prefer the REST Team Chat skill for bulk operations, durable retry logic, approval workflows, or audit-heavy production systems.

## Available Tools

The current Team Chat MCP tool surface is:

- `zoom_chat_message_send`
- `zoom_chat_message_update`
- `zoom_chat_contact_add`
- `zoom_chat_channel_create`
- `zoom_chat_channel_update`
- `zoom_chat_channel_members_add`

Some MCP clients namespace server tools in the UI. Treat the raw tool names above as
authoritative after `tools/list`.

Reference: [references/tools.md](references/tools.md)

## Common Workflows

**Send a message:**

```text
zoom_chat_message_send
  chat_session_id: "CHANNEL_ID_OR_USER_ID_OR_EMAIL"
  message_content: "Deployment starts at 17:00 UTC."
  message_format: "text"
```

**Send a threaded reply:**

```text
zoom_chat_message_send
  chat_session_id: "CHANNEL_ID"
  parent_message_id: "PARENT_MESSAGE_ID"
  message_content: "Confirmed. I will post the follow-up here."
  message_format: "text"
```

**Edit a message sent by the caller:**

```text
zoom_chat_message_update
  chat_session_id: "CHANNEL_ID_OR_USER_ID"
  messageId: "MESSAGE_ID"
  message_content: "Updated status: deployment completed."
  message_format: "text"
```

**Create a channel:**

```text
zoom_chat_channel_create
  channel_name: "incident-response"
  channel_type: "private_channel"
  post_message_permission: "everyone"
  mention_all_permission: "channel_owner_and_admin"
  new_members_can_see_previous_messages_and_files: true
```

**Add members to a channel:**

```text
zoom_chat_channel_members_add
  channelId: "CHANNEL_ID"
  user_email_list:
    - "person@example.com"
```

## Chaining

- Parent MCP skill: [../SKILL.md](../SKILL.md)
- Deterministic Team Chat API skill: [../../team-chat/SKILL.md](../../team-chat/SKILL.md)
- OAuth guidance: [../concepts/oauth-setup.md](../concepts/oauth-setup.md)
- General routing: [../../general/SKILL.md](../../general/SKILL.md)

## References

- [references/tools.md](references/tools.md) - Team Chat MCP tool catalog, parameters, scopes, and constraints.
- Zoom docs: https://developers.zoom.us/docs/mcp/zoom-chat-mcp-server/
