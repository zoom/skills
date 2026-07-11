---
title: "Meeting SDK Signature Playbook"
---

# Meeting SDK Signature Playbook

Most “join failed” issues reduce to signature generation or mismatched inputs.

## Rules

1. **Generate signatures server-side only**. Never ship the Client Secret to the browser/app.
2. `meetingNumber` must be digits only.
3. `role` must match what you are doing:
   - `0` = join as attendee
   - `1` = start as host
4. Keep `iat/exp` reasonable and account for clock skew (server time matters).

## Required Payload Fields (Common Pattern)

You’ll typically see fields like:

- `appKey` (Meeting SDK Client ID)
- `mn` (meeting number)
- `role`
- `iat`, `exp`, `tokenExp`

If the developer is mixing in REST API OAuth tokens or Marketplace JWT app-type tokens, stop and clarify: those are **not** Meeting SDK signatures.

## Common Failure Modes

- **Invalid signature**:
  - wrong Client ID/Client Secret pair
  - missing `appKey`
  - wrong `mn` format
  - expired `exp/tokenExp`
  - generating signature for role=1 but joining (or vice versa)
- **4003 Invalid Parameter** (common on Web “start” flows):
  - role mismatch or missing host requirements (often needs ZAK for host start flows)
- **Works locally but not in prod**:
  - different env vars/secret
  - prod server clock skew

For Web SDK `4.0.0+`, omit the deprecated `sdkKey` field from `join()`. Pass the signed JWT in
`signature`; do not add a literal prefix outside the JWT.

## Web-Specific Gotcha: `passWord`

On Web Client View (`ZoomMtg.join`) the key is `passWord` (capital W). If the meeting has a passcode and it’s missing/misnamed, join fails in a way that looks like auth trouble.
