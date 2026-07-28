# Slash Commands (Chatbot API)

Slash commands are configured on the Marketplace app and trigger webhook events.

## Pattern

1. Configure `/yourcommand` in the Chatbot feature settings.
2. User runs the command in Team Chat.
3. Your webhook receives `bot_notification` (or equivalent) with the command text.
4. Confirm `cmd`, `toJid`, `userJid`, and `accountId` are present.
5. Get a chatbot token with `grant_type=client_credentials` and respond with a
   message card using `robot_jid`, `to_jid`, `user_jid`, and `account_id`.
6. Inspect the outbound API response and verify the reply in Team Chat.

## Pitfalls

- Commands are account-scoped; make sure you're testing in the right account.
- Don’t rely on client-side parsing; parse on your server.
- A webhook HTTP 200 only acknowledges receipt; it does not confirm delivery of
  the bot reply.
