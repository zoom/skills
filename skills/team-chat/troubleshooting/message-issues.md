# Message Issues

## Messages Not Sending

- Confirm you're using the correct API:
  - Team Chat API uses user OAuth token
  - Chatbot API uses a `client_credentials` token + `robot_jid`
- Never use an authorization-code/user OAuth token with `/v2/im/chat/messages`.
- Include `robot_jid`, `to_jid`, `user_jid`, `account_id`, and `content.body`.
- Inspect and log the outbound API status and response body.
- A webhook HTTP 200 only confirms event receipt, not chatbot reply success.

## Card Not Rendering

- Validate the card JSON payload against known-good examples.
- Simplify to a minimal card and add components incrementally.
