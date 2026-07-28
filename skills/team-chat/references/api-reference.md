# API Reference Pointers

This doc is intentionally lightweight; prefer the official REST reference for the authoritative schema.

## Team Chat API (user-level)

- Send message: `POST /v2/chat/users/me/messages`
- Typical needs:
  - list channels
  - post to channel / DM
  - thread replies

## Chatbot API (bot-level)

- Send bot message: `POST /v2/im/chat/messages`
- Authenticate with `grant_type=client_credentials`; do not send a user OAuth token.
- Minimum reply fields from a `bot_notification` payload:
  - `robot_jid` (configured Bot JID)
  - `to_jid` (`payload.toJid`)
  - `user_jid` (`payload.userJid`)
  - `account_id` (`payload.accountId`)
  - `content.body` with one or more message components
- Inspect the outbound HTTP status and body. A webhook HTTP 200 only confirms
  that Zoom delivered the event to your webhook.

## Notes

- If you see "invalid access token" errors, check:
  - app type (General App OAuth vs others)
  - scopes
  - whether the user re-consented after scope changes
