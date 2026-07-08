# Authentication

Authentication methods for Zoom APIs and SDKs.

## Overview

Zoom supports multiple authentication methods depending on your use case:

| Method | Use Case |
|--------|----------|
| **General App OAuth 2.0** | User-level or admin/account-level authorization through a General App |
| **General App client credentials** | App-owned access for selected scopes such as Team Chat bot and Marketplace app operations |
| **Server-to-Server OAuth** | Server-side automation (no user interaction) |
| **SDK JWT** | Meeting SDK and Video SDK authentication |

## General App OAuth 2.0

For General Apps that act on behalf of an installing user or an admin-authorized account.
General Apps can be user-level scoped or admin/account-level scoped.

### Flow

```
1. User clicks "Connect with Zoom"
2. Redirect to Zoom authorization URL
3. User grants permission
4. Zoom redirects back with auth code
5. Exchange code for access token
6. Use token to call APIs
```

### Authorization URL

```
https://zoom.us/oauth/authorize?response_type=code&client_id={clientId}&redirect_uri={redirectUri}
```

### Token Exchange

```bash
curl -X POST "https://zoom.us/oauth/token" \
  -H "Authorization: Basic {base64(clientId:clientSecret)}" \
  -d "grant_type=authorization_code&code={authCode}&redirect_uri={redirectUri}"
```

## Server-to-Server OAuth

For server-side automation without user interaction.

### Get Access Token

```bash
curl -X POST "https://zoom.us/oauth/token?grant_type=account_credentials&account_id={accountId}" \
  -H "Authorization: Basic {base64(clientId:clientSecret)}"
```

### Response

```json
{
  "access_token": "eyJ...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

## General App Client Credentials

For selected app-owned scopes, use the General App Client ID and Client Secret
from **App Credentials** to request a token without `account_id`.

```bash
curl -X POST "https://zoom.us/oauth/token?grant_type=client_credentials" \
  -H "Authorization: Basic {base64(clientId:clientSecret)}"
```

Use this only when the endpoint requires app-owned scopes, such as selected
Marketplace event-subscription or WebSocket operations.

## SDK JWT Signatures

For Meeting SDK and Video SDK authentication. See:
- [Meeting SDK Authorization](../../meeting-sdk/references/authorization.md)
- [Video SDK Authorization](../../video-sdk/references/authorization.md)

### Best Practices

| Practice | Recommendation |
|----------|----------------|
| **Expiry (`exp`)** | Set ~10 seconds after generation |
| **Issued At (`iat`)** | Set 2 hours in the past (if `exp - iat >= 2 hours` required) |
| **Generate server-side** | Never expose secrets in client code |

## Resources

- **OAuth docs**: https://developers.zoom.us/docs/integrations/oauth/
- **S2S OAuth docs**: https://developers.zoom.us/docs/internal-apps/s2s-oauth/
