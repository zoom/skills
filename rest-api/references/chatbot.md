# Zoom Chatbot API

Build interactive bots for Zoom Team Chat.

## Overview

Zoom Chatbots are apps that respond to messages in Team Chat. They can:
- Respond to slash commands
- Send interactive messages with buttons
- Process user input
- Integrate external services

## How Chatbots Work

```
User types: /weather 94102
     ↓
Zoom sends webhook to your server
     ↓
Your server processes request
     ↓
Server responds via API
     ↓
Bot replies in chat
```

## Creating a Chatbot

1. Go to [Zoom App Marketplace](https://marketplace.zoom.us/)
2. Click **Develop** → **Build App**
3. Select **Team Chat Apps** → **Chatbot**
4. Configure:
   - Bot name and description
   - Slash command (e.g., `/mybot`)
   - Webhook URL
   - OAuth scopes

## Webhook Events

### Bot Mentioned/Command

When user sends `/yourbot` command:

```json
{
  "event": "bot_notification",
  "payload": {
    "accountId": "abc123",
    "channelName": "general",
    "cmd": "/yourbot help",
    "robotJid": "yourbot@xmpp.zoom.us",
    "toJid": "channel_jid@xmpp.zoom.us",
    "userJid": "sender_jid@xmpp.zoom.us",
    "userName": "John Doe",
    "userId": "user_123",
    "timestamp": 1705312200000
  }
}
```

### Interactive Message Action

When user clicks button:

```json
{
  "event": "interactive_message_actions",
  "payload": {
    "accountId": "abc123",
    "actionItem": {
      "text": "Approve",
      "value": "approve_123"
    },
    "messageId": "msg_xyz",
    "channelId": "channel_123",
    "robotJid": "yourbot@xmpp.zoom.us",
    "userJid": "sender_jid@xmpp.zoom.us"
  }
}
```

## Sending Messages

### Basic Text Response

```javascript
// POST /im/chat/messages
await fetch(
  'https://api.zoom.us/v2/im/chat/messages',
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      robot_jid: 'yourbot@xmpp.zoom.us',
      to_jid: payload.toJid,
      account_id: payload.accountId,
      content: {
        head: {
          text: 'Bot Response'
        },
        body: [
          {
            type: 'message',
            text: 'Hello! How can I help you?'
          }
        ]
      }
    })
  }
);
```

### Interactive Message with Buttons

```javascript
await fetch(
  'https://api.zoom.us/v2/im/chat/messages',
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      robot_jid: 'yourbot@xmpp.zoom.us',
      to_jid: payload.toJid,
      account_id: payload.accountId,
      content: {
        head: {
          text: 'Approval Request',
          sub_head: {
            text: 'Request #12345'
          }
        },
        body: [
          {
            type: 'message',
            text: 'John Doe is requesting access to Project Alpha.'
          },
          {
            type: 'actions',
            items: [
              {
                text: 'Approve',
                value: 'approve_12345',
                style: 'Primary'
              },
              {
                text: 'Deny',
                value: 'deny_12345',
                style: 'Danger'
              }
            ]
          }
        ]
      }
    })
  }
);
```

### Message with Form Fields

```javascript
await fetch(
  'https://api.zoom.us/v2/im/chat/messages',
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      robot_jid: 'yourbot@xmpp.zoom.us',
      to_jid: payload.toJid,
      account_id: payload.accountId,
      content: {
        head: {
          text: 'Create Ticket'
        },
        body: [
          {
            type: 'fields',
            items: [
              {
                key: 'Title',
                value: '',
                editable: true
              },
              {
                key: 'Priority',
                value: 'Medium',
                editable: true,
                select: {
                  items: [
                    { text: 'Low', value: 'low' },
                    { text: 'Medium', value: 'medium' },
                    { text: 'High', value: 'high' }
                  ]
                }
              }
            ]
          },
          {
            type: 'actions',
            items: [
              {
                text: 'Submit',
                value: 'submit_ticket',
                style: 'Primary'
              }
            ]
          }
        ]
      }
    })
  }
);
```

## Message Elements

### Head (Header)

```javascript
head: {
  text: 'Main Title',
  sub_head: {
    text: 'Subtitle'
  },
  style: {
    bold: true,
    color: '#1E90FF'
  }
}
```

### Body Elements

| Type | Description |
|------|-------------|
| `message` | Text content |
| `actions` | Button row |
| `fields` | Key-value pairs / form inputs |
| `attachment` | File/image |
| `section` | Grouped content |

### Actions (Buttons)

```javascript
{
  type: 'actions',
  items: [
    {
      text: 'Button Label',
      value: 'action_value',  // Returned in webhook
      style: 'Primary'        // Primary, Danger, Default
    }
  ]
}
```

### Fields

```javascript
{
  type: 'fields',
  items: [
    {
      key: 'Status',
      value: 'Active',
      short: true  // Side-by-side with another short field
    },
    {
      key: 'Owner',
      value: 'John Doe',
      short: true
    }
  ]
}
```

## Handling Button Clicks

```javascript
// Express.js webhook handler
app.post('/webhook', async (req, res) => {
  const { event, payload } = req.body;
  
  if (event === 'interactive_message_actions') {
    const action = payload.actionItem.value;
    
    if (action.startsWith('approve_')) {
      const requestId = action.split('_')[1];
      await approveRequest(requestId);
      
      // Update the original message
      await updateBotMessage(payload.messageId, {
        content: {
          head: { text: 'Request Approved' },
          body: [{ type: 'message', text: `Approved by ${payload.userName}` }]
        }
      });
    }
  }
  
  res.sendStatus(200);
});
```

## Update Existing Message

```javascript
// PATCH /im/chat/messages/{messageId}
await fetch(
  `https://api.zoom.us/v2/im/chat/messages/${messageId}`,
  {
    method: 'PATCH',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      robot_jid: 'yourbot@xmpp.zoom.us',
      account_id: accountId,
      content: {
        head: { text: 'Updated Message' },
        body: [{ type: 'message', text: 'This message was updated!' }]
      }
    })
  }
);
```

## Delete Message

```javascript
// DELETE /im/chat/messages/{messageId}
await fetch(
  `https://api.zoom.us/v2/im/chat/messages/${messageId}?robot_jid=yourbot@xmpp.zoom.us&account_id=${accountId}`,
  {
    method: 'DELETE',
    headers: { 'Authorization': `Bearer ${accessToken}` }
  }
);
```

## Slash Commands

Configure slash commands in Marketplace:

| Field | Example |
|-------|---------|
| Command | `/weather` |
| Description | Get weather for a location |
| Hint | `[city or zip code]` |
| Scopes | `imchat:bot` |

User types: `/weather San Francisco`

Your webhook receives:
```json
{
  "cmd": "/weather San Francisco"
}
```

Parse and respond:
```javascript
const [command, ...args] = payload.cmd.split(' ');
const location = args.join(' ');
const weather = await getWeather(location);

await sendBotMessage(payload.toJid, {
  head: { text: `Weather in ${location}` },
  body: [{ type: 'message', text: `${weather.temp}°F, ${weather.condition}` }]
});
```

## Required Scopes

| Scope | Description |
|-------|-------------|
| `imchat:bot` | Send chatbot messages |
| `imchat:bot:read` | Read chatbot interactions |

## Best Practices

1. **Respond quickly** - Acknowledge within 3 seconds, process async
2. **Use ephemeral messages** - For error messages only the user should see
3. **Update instead of spam** - Update existing messages instead of sending new ones
4. **Handle errors gracefully** - Show friendly error messages
5. **Provide help** - Respond to `/yourbot help` with usage instructions

## Example: Approval Bot

```javascript
// Webhook handler
app.post('/webhook', async (req, res) => {
  const { event, payload } = req.body;
  
  // Acknowledge immediately
  res.sendStatus(200);
  
  if (event === 'bot_notification') {
    const [cmd, action, ...args] = payload.cmd.split(' ');
    
    if (action === 'request') {
      // Create approval request
      const requestId = await createRequest(payload.userId, args.join(' '));
      
      await sendApprovalCard(payload.toJid, requestId, payload.userName);
    } else if (action === 'help') {
      await sendHelpMessage(payload.toJid);
    }
  }
  
  if (event === 'interactive_message_actions') {
    const [action, requestId] = payload.actionItem.value.split('_');
    
    if (action === 'approve') {
      await approveRequest(requestId, payload.userId);
      await updateMessageWithResult(payload.messageId, 'Approved', payload.userName);
    } else if (action === 'deny') {
      await denyRequest(requestId, payload.userId);
      await updateMessageWithResult(payload.messageId, 'Denied', payload.userName);
    }
  }
});
```

## Resources

- **Chatbot docs**: https://developers.zoom.us/docs/zoom-team-chat-apps/
- **Message format**: https://developers.zoom.us/docs/zoom-team-chat-apps/create-a-chatbot-app/
- **Sample bots**: https://github.com/zoom/chatbot-sample
