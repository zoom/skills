# Zoom Team Chat API

Send messages, manage channels, and integrate with Zoom's Team Chat.

## Overview

Zoom Team Chat (formerly Zoom Chat) is Zoom's messaging platform. The API allows you to:
- Send messages to users and channels
- Create and manage channels
- Share files
- Manage contact lists
- Build chat integrations

## Base URL

```
https://api.zoom.us/v2/chat
```

## Authentication

Use OAuth 2.0 with Team Chat scopes. See `general/references/authentication.md`.

## Core Endpoints

### Messages

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/chat/users/{userId}/messages` | Send message |
| GET | `/chat/users/{userId}/messages` | List messages |
| PUT | `/chat/users/{userId}/messages/{messageId}` | Update message |
| DELETE | `/chat/users/{userId}/messages/{messageId}` | Delete message |

### Channels

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/chat/users/{userId}/channels` | List user's channels |
| POST | `/chat/users/{userId}/channels` | Create channel |
| GET | `/chat/channels/{channelId}` | Get channel details |
| PATCH | `/chat/channels/{channelId}` | Update channel |
| DELETE | `/chat/channels/{channelId}` | Delete channel |
| GET | `/chat/channels/{channelId}/members` | List members |
| POST | `/chat/channels/{channelId}/members` | Add members |
| DELETE | `/chat/channels/{channelId}/members/{memberId}` | Remove member |

### Contacts

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/chat/users/{userId}/contacts` | List contacts |
| GET | `/chat/users/{userId}/contacts/{contactId}` | Get contact |

## Common Operations

### Send Message to User

```javascript
// POST /chat/users/{userId}/messages
const response = await fetch(
  `https://api.zoom.us/v2/chat/users/me/messages`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      to_jid: 'user_jid@xmpp.zoom.us',  // Recipient JID
      message: 'Hello from the API!'
    })
  }
);
```

### Send Message to Channel

```javascript
// POST /chat/users/{userId}/messages
await fetch(
  `https://api.zoom.us/v2/chat/users/me/messages`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      to_channel: 'channel_id_123',
      message: 'Team announcement!'
    })
  }
);
```

### Send Rich Message (with formatting)

```javascript
await fetch(
  `https://api.zoom.us/v2/chat/users/me/messages`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      to_channel: 'channel_id_123',
      message: 'New deployment completed!',
      rich_text: [
        {
          start_position: 0,
          end_position: 3,
          format_type: 'Bold'
        }
      ],
      at_items: [
        {
          at_type: 1,  // 1 = @mention user
          at_contact: 'user_jid@xmpp.zoom.us',
          start_position: 25,
          end_position: 30
        }
      ]
    })
  }
);
```

### Create Channel

```javascript
// POST /chat/users/{userId}/channels
const channel = await fetch(
  `https://api.zoom.us/v2/chat/users/me/channels`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      name: 'Project Alpha',
      type: 1,  // 1 = private, 2 = public
      members: [
        { email: 'user1@example.com' },
        { email: 'user2@example.com' }
      ]
    })
  }
).then(r => r.json());

// { "id": "channel_123", "jid": "channel_jid@xmpp.zoom.us", "name": "Project Alpha" }
```

### List Messages in Channel

```javascript
// GET /chat/users/{userId}/messages
const params = new URLSearchParams({
  to_channel: 'channel_id_123',
  from: '2024-01-01T00:00:00Z',
  to: '2024-01-31T23:59:59Z',
  page_size: 50
});

const messages = await fetch(
  `https://api.zoom.us/v2/chat/users/me/messages?${params}`,
  {
    headers: { 'Authorization': `Bearer ${accessToken}` }
  }
).then(r => r.json());
```

### Share File

```javascript
// POST /chat/users/{userId}/messages/files
const formData = new FormData();
formData.append('to_channel', 'channel_id_123');
formData.append('files', fileBlob, 'document.pdf');

await fetch(
  `https://api.zoom.us/v2/chat/users/me/messages/files`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`
    },
    body: formData
  }
);
```

## Message Types

| Type | Description |
|------|-------------|
| Text | Plain text message |
| File | File attachment |
| Image | Image attachment |
| Code snippet | Code block with syntax highlighting |
| Interactive | Message with action buttons (Chatbot only) |

## Channel Types

| Type | Value | Description |
|------|-------|-------------|
| Private | 1 | Invite-only channel |
| Public | 2 | Anyone in org can join |
| Member | 3 | Direct message (1:1) |
| Group | 4 | Group chat (no channel) |

## Webhooks

Team Chat events via webhooks:

| Event | Trigger |
|-------|---------|
| `chat_message.sent` | New message sent |
| `chat_message.updated` | Message edited |
| `chat_message.deleted` | Message deleted |
| `chat_channel.created` | New channel created |
| `chat_channel.updated` | Channel updated |
| `chat_channel.deleted` | Channel deleted |
| `chat_channel.member_added` | Member joined |
| `chat_channel.member_removed` | Member left |

### Webhook Payload Example

```json
{
  "event": "chat_message.sent",
  "payload": {
    "account_id": "abc123",
    "object": {
      "id": "msg_xyz",
      "type": "to_channel",
      "channel_id": "channel_123",
      "sender": "user_jid@xmpp.zoom.us",
      "message": "Hello team!",
      "date_time": "2024-01-15T10:30:00Z"
    }
  }
}
```

## Required Scopes

| Scope | Description |
|-------|-------------|
| `chat_message:read` | Read messages |
| `chat_message:write` | Send messages |
| `chat_channel:read` | Read channels |
| `chat_channel:write` | Create/manage channels |
| `chat_contact:read` | Read contacts |
| `chat_message:read:admin` | Read all messages (admin) |
| `chat_message:write:admin` | Send as any user (admin) |

## Limitations

| Limitation | Value |
|------------|-------|
| Message length | 4096 characters |
| File size | 512 MB |
| Members per channel | 10,000 |
| Channels per user | 500 |

## Use Cases

- **Notifications**: Send alerts to channels
- **Onboarding bots**: Welcome new team members
- **Incident management**: Create channels for incidents
- **Workflow automation**: Post updates from CI/CD
- **Integration hub**: Connect external tools to Team Chat

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/team-chat/methods/
- **Chatbots**: See `chatbot.md` for interactive messages
