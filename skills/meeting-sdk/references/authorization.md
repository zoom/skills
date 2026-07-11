# Meeting SDK - Authorization

Generate JWT signatures for Meeting SDK authentication.

## Overview

Meeting SDK uses JWT (JSON Web Token) signatures to authenticate users joining meetings. Signatures must be generated server-side to protect your Client Secret.

## Prerequisites

- Meeting SDK Client ID and Client Secret from [Marketplace](https://marketplace.zoom.us/) (sign-in required)
- Server-side code to generate signatures

## JWT Structure

| Claim | Description |
|-------|-------------|
| `appKey` | Your Meeting SDK Client ID |
| `mn` | Meeting number |
| `role` | 0 = participant, 1 = host |
| `iat` | Issued at timestamp |
| `exp` | Expiration timestamp |
| `tokenExp` | Token expiration timestamp |

## Signature Generation Best Practices

### Short-Lived Tokens (Recommended)

For security, generate tokens with short expiry:

```javascript
const iat = Math.floor(Date.now() / 1000) - 30; // tolerate small clock skew
const exp = iat + 60 * 60 * 2;                  // two hours

const payload = {
  appKey: CLIENT_ID,
  mn: meetingNumber,
  role: role,
  iat: iat,
  exp: exp,
  tokenExp: exp
};
```

The current auth reference requires `exp` and `tokenExp` to be at least 1,800 seconds after
`iat` and recommends no more than 48 hours. Generate on demand and enforce authorization at your
signature endpoint rather than manipulating `iat` far into the past.

### Server-Side Example (Node.js)

```javascript
const jwt = require('jsonwebtoken');

function generateSignature(clientId, clientSecret, meetingNumber, role) {
  const iat = Math.floor(Date.now() / 1000) - 30;
  const exp = iat + 60 * 60 * 2;
  
  const payload = {
    appKey: clientId,
    mn: meetingNumber,
    role: role,
    iat: iat,
    exp: exp,
    tokenExp: exp
  };
  
  return jwt.sign(payload, clientSecret, { algorithm: 'HS256' });
}
```

## Role Values

| Role | Value | Description |
|------|-------|-------------|
| Participant | 0 | Join as attendee |
| Host | 1 | Join as host (requires host key or being meeting owner) |

## Security Guidelines

| Do | Don't |
|----|-------|
| Generate signatures server-side | Expose Client Secret in client code |
| Use short expiry times | Use long-lived tokens |
| Validate user before generating | Generate for unauthenticated users |

## Resources

- **Auth docs**: https://developers.zoom.us/docs/meeting-sdk/auth/
