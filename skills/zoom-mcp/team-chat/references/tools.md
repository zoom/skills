# Tools — Team Chat MCP

Current catalog for `https://mcp.zoom.us/mcp/chat/streamable`. The older
`/mcp/team_chat/streamable` route remains a runtime alias.

This is a read/write MCP surface for Zoom Team Chat. It is separate from the default Zoom MCP
server's broader `search_zoom` tool.

## Tool Catalog

| Tool | Purpose | Required scope |
|------|---------|----------------|
| `zoom_chat_message_send` | Send a message to a 1:1 chat or channel; optionally reply in a thread | `team_chat:write:user_message` |
| `zoom_chat_message_update` | Edit an existing message previously sent by the caller | `team_chat:update:user_message` |
| `zoom_chat_contact_add` | Send a contact invitation by email | `team_chat:write:contact_information` |
| `zoom_chat_channel_create` | Create a public channel, private channel, or group chat | `team_chat:write:user_channel` |
| `zoom_chat_channel_update` | Update channel settings such as name, visibility, and permissions | `team_chat:update:user_channel` |
| `zoom_chat_channel_members_add` | Add users to an existing channel by email | `team_chat:write:members` |
| `zoom_chat_channel_get_by_id` | Get a channel by ID | `team_chat:read:channel` |
| `zoom_chat_channel_member_role_update` | Update a channel member's role | `team_chat:update:channel_member_role` |
| `zoom_chat_channel_members_list` | List channel members | `team_chat:read:list_members` |
| `zoom_chat_channels_list` | List the caller's channels | `team_chat:read:list_user_channels` |
| `zoom_chat_channels_search` | Search channels by keyword, ownership, or visibility | `team_chat:read:list_user_channels` |
| `zoom_chat_contacts_get_by_id` | Resolve one or more contacts by user/member ID | `team_chat:read:list_contacts` |
| `zoom_chat_contacts_search` | Search contacts by name or email | `team_chat:read:list_contacts` |
| `zoom_chat_files_search` | Search shared Team Chat files | `team_chat:read:list_user_files` |
| `zoom_chat_message_get_by_id` | Get a message from a chat session | `team_chat:read:list_user_messages` |
| `zoom_chat_message_replies_list` | List replies in a message thread | `team_chat:read:list_user_messages` |
| `zoom_chat_messages_fetch` | Fetch messages from a chat session | `team_chat:read:list_user_messages` |
| `zoom_chat_messages_filter` | Filter messages such as unread, mentions, or sent-by-me | `team_chat:read:list_user_messages` |
| `zoom_chat_messages_search` | Search messages in one or all sessions | `team_chat:read:list_user_messages` |
| `zoom_chat_sessions_recent_list` | List recent channels and direct-message sessions | `team_chat:read:list_user_sessions` |

## Supported Scope Families

Protected-resource metadata for Team Chat MCP advertises:

- `team_chat:write:user_message`
- `team_chat:update:user_message`
- `team_chat:write:contact_information`
- `team_chat:write:user_channel`
- `team_chat:update:user_channel`
- `team_chat:write:members`
- `team_chat:read:channel`
- `team_chat:update:channel_member_role`
- `team_chat:read:list_members`
- `team_chat:read:list_user_channels`
- `team_chat:read:list_contacts`
- `team_chat:read:list_user_files`
- `team_chat:read:list_user_messages`
- `team_chat:read:list_user_sessions`
- `chat_channel:read`
- `chat_channel:write`

Metadata endpoint:

```text
https://mcp.zoom.us/.well-known/oauth-protected-resource/mcp_chat
```

## Message Tools

### `zoom_chat_message_send`

Send a Zoom Team Chat message to a direct chat or channel.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `chat_session_id` | string | **Yes** | Target session. Use user/member ID or user email for 1:1 messages, or channel ID for channel messages. |
| `message_content` | string | **Yes** | Message body. Cannot be empty. |
| `message_format` | string | No | `text` or `markdown`; default is `text`. |
| `parent_message_id` | string | No | Parent message ID for a threaded reply. |

Notes:
- Markdown mode supports standard Markdown and Zoom Chat extended markup.
- Tables are not supported.
- For replies to replies, use the parent message ID rather than nesting further.

### `zoom_chat_message_update`

Edit an existing message previously sent by the caller.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `chat_session_id` | string | **Yes** | Session where the message exists. |
| `messageId` | string | **Yes** | Target message ID. |
| `message_content` | string | **Yes** | Replacement message body. Cannot be empty. |
| `message_format` | string | No | `text` or `markdown`; default is `text`. |

Constraints:
- Only the original sender can edit their own messages.
- The message must exist and be accessible to the caller.

## Contact Tool

### `zoom_chat_contact_add`

Send a contact invitation by email.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `invitee_email` | string | **Yes** | Email address of the target Zoom user. |
| `message` | string | No | Optional invite note; maximum length is 150 characters. |

Notes:
- The recipient must have a Zoom account associated with the email address.
- Rate limits and anti-spam limits apply.

## Channel Tools

### `zoom_chat_channel_create`

Create a new Team Chat channel.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `channel_name` | string | **Yes** | Unique channel name within the organization; 1-100 characters. |
| `channel_description` | string | No | Optional channel purpose/usage description. |
| `channel_type` | string | No | `public_channel`, `private_channel`, or `group_chat`; default is `public_channel`. |
| `post_message_permission` | string | No | `everyone`, `channel_owner_and_admin`, or `nobody`; default is `everyone`. |
| `mention_all_permission` | string | No | `everyone`, `channel_owner_and_admin`, or `disable`; default is `everyone`. |
| `new_members_can_see_previous_messages_and_files` | boolean | No | Default is `true`. |

### `zoom_chat_channel_update`

Update an existing channel.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `channelId` | string | **Yes** | Target channel ID. |
| `channel_name` | string | No | New unique channel name. |
| `channel_type` | string | No | `public_channel` or `private_channel`; group chat conversion is not allowed. |
| `post_message_permission` | string | No | `everyone`, `channel_owner_and_admin`, or `nobody`. |
| `mention_all_permission` | string | No | `everyone`, `channel_owner_and_admin`, or `disable`. |
| `new_members_can_see_previous_messages_and_files` | boolean | No | Whether newly added members can see prior messages/files. |

Constraints:
- Only provided fields are updated.
- Only channel owner/admin can update channel settings.
- Group chats cannot be converted through `channel_type`.

### `zoom_chat_channel_members_add`

Add one or more users to a channel by email address.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `channelId` | string | **Yes** | Target channel ID. |
| `user_email_list` | array | No | User email addresses to add; maximum recommended batch is 100 users. |

Notes:
- Users who are already members are automatically filtered out.
- The response separates successfully added user IDs and emails.

## Routing Notes

- Use `zoom-mcp/team-chat` for agent-driven read/search/write/update Team Chat actions.
- Use the default `zoom-mcp` `search_zoom` tool for read-only Team Chat and Zoom Docs search.
- Use `team-chat` REST skill for deterministic production integrations, bulk jobs, webhooks, and custom retry/audit requirements.
