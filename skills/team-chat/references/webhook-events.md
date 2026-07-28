# Webhook Events (Chatbot API)

Common webhook event types you will handle:

- `bot_notification`: user messages your bot or triggers a command
- `interactive_message_actions`: user clicks a button
- `chat_message.submit`: user submits a form
- `bot_installed`: bot added to an account
- `app_deauthorized`: bot removed / app deauthorized

For `bot_notification` and interactive action replies, preserve `toJid`,
`userJid`, and `accountId` from the incoming payload. Use them as `to_jid`,
`user_jid`, and `account_id` in `POST /v2/im/chat/messages`; do not substitute
values from another account or Marketplace environment.

## Handler Checklist

- Verify the request (per Zoom's verification guidance).
- Parse payload carefully (treat as untrusted input).
- Route by event type and action values.
- Respond quickly; do heavy work async if needed.
- Treat webhook HTTP 200 as event receipt only. Inspect the outbound message
  API response and verify the reply in Team Chat.
