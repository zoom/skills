# App Types

Choose the right current Zoom Marketplace app type for your integration.

## Overview

Zoom Marketplace currently has 3 primary app types:

| App Type | Use Case |
|----------|----------|
| **General App** | User-installed or admin-installed app with OAuth, embeds, surfaces, webhooks, Zoom Apps, chatbot, or app-owned client credentials |
| **Server-to-Server OAuth** | Internal backend automation against your own account without user authorization |
| **Webhook-only App** | Receive events only, with no OAuth API access |

Legacy app types such as OAuth, Chatbot, and Zoom Apps are now represented inside the
**General App** build flow. When older docs or forum answers say "OAuth app", map that
to **General App with OAuth enabled**.

## General App

The modular app type. Use it when the app acts on behalf of a user or account,
needs install/authorization, uses Zoom client surfaces, or combines multiple features.

### Scope Level

General Apps can be configured with either user-level or admin/account-level scopes:

| General App scope level | Scopes | Token context | Typical use |
|-------------------------|--------|---------------|-------------|
| **User-level scoped** | User scopes, usually no `:admin` suffix | Acts as the installing user | Self-service SaaS, user meeting/calendar/chat integrations |
| **Admin/account-level scoped** | Admin scopes, commonly `*:admin` granular scopes | Acts with account-level admin authorization | Enterprise installs, account dashboards, managing users across the account |

Implementation rules:
- User-level General App tokens usually target the current user and commonly use `me` in user paths.
- Admin/account-level General App tokens can access account resources allowed by the granted admin scopes.
- Scope changes require reauthorization for existing installations before tokens include the new scopes.
- Some app-owned operations, such as selected Marketplace event-subscription and WebSocket scopes, use `grant_type=client_credentials` from a General App rather than user authorization.

Marketplace API manifest values:
- User-level General App manifests use `oauth_information.usage: "USER_OPERATION"`.
- Admin/account-level General App manifests use `oauth_information.usage: "ADMIN_MANAGEMENT"`.
- `oauth_information.scopes` must match the usage mode; user mode rejects admin scopes and admin mode rejects user scopes.
- App-owned Marketplace scopes are not placed in `oauth_information.scopes`; use General App `client_credentials` when the endpoint requires app-owned access.
- For exact create/get/delete response shapes and live quirks, see [Marketplace App Management](../../rest-api/references/marketplace-apps.md).
- For observed manifest archetypes across Meeting SDK, RTMS, ZCC/Phone, Team Chat, Admin OAuth, Chatbot, webhook-heavy, and MCP connector apps, see [Existing App Manifest Archetypes](../../rest-api/references/marketplace-apps.md#existing-app-manifest-archetypes-observed).
- Manifest export is not universal: draft General apps with granular scopes export cleanly, but S2S/private OAuth-style and older/non-granular apps can fail export. See [Export caveats](../../rest-api/references/marketplace-apps.md#manifest-export-caveats).

### App-Owned Client Credentials

When an endpoint requires app-owned scopes, use the **Client ID** and **Client Secret**
from the General App's **App Credentials** page to request a `client_credentials` token.

```bash
curl -X POST "https://zoom.us/oauth/token?grant_type=client_credentials" \
  -H "Authorization: Basic $(printf '%s:%s' "$ZOOM_CLIENT_ID" "$ZOOM_CLIENT_SECRET" | base64)" \
  -H "Content-Type: application/x-www-form-urlencoded"
```

Use this token only for scopes/endpoints that explicitly require app-owned access,
such as selected Marketplace event-subscription and WebSocket operations. Do not add
`account_id`; that belongs to Server-to-Server OAuth.

### Surfaces (product contexts)

Your app can interact with these Zoom products:

- Meetings
- Webinars
- Rooms
- Phone
- Team Chat
- Contact Center
- Whiteboard
- Virtual Agent
- Events
- Mail
- Workflows

### Embeds (SDKs)

Embed Zoom functionality in your app:

| Embed | Description |
|-------|-------------|
| **Meeting SDK** | Embed Zoom meetings |
| **Contact Center SDK** | Embed Contact Center |
| **Phone SDK** | Embed Phone functionality |

### Access

Configure in the Access tab:

- **Secret Token** - Verify webhook notifications
- **Event Subscription** - Webhooks
- **WebSockets** - Real-time event connections

### Scopes

Define which API methods the app can call. Scopes are:
- Restricted to specific resources
- Reviewed by Zoom during app submission

### Features in General App

General App can also include:
- **Zoom Apps** - Apps that run inside Zoom client
- **Chatbot** - Team Chat bot capabilities
- **Webhooks** - Event subscriptions alongside OAuth/API access
- **WebSockets** - App-owned event streaming where supported

## Server-to-Server OAuth

Backend automation without user authorization.

- No user interaction required
- Access your account's data
- Can include webhooks and zoom-websockets
- Best for: automation, reporting, integrations
- Marketplace API creation for S2S apps is expected around 2026-07-12; before that
  rollout, create S2S apps through Marketplace UI and use API testing only for General Apps.

## Webhook-only App

Event notifications only.

- Receive events, no API calls
- No OAuth tokens needed
- Best for: event logging, triggering external workflows

Use this when you only need events. Otherwise, add webhooks to a General App or S2S app.

## Decision Guide

| Need | App Type |
|------|----------|
| Call APIs for your account (backend) | Server-to-Server OAuth |
| Call APIs on behalf of a user | General App with user-level scopes |
| Call APIs across an account after admin install | General App with admin/account-level scopes |
| Embed Zoom meetings | General App + Meeting SDK embed |
| Embed Contact Center | General App + Contact Center SDK embed |
| Embed Phone | General App + Phone SDK embed |
| Build in-client app | General App + Zoom Apps |
| Build a Team Chat bot | General App + Chatbot capability |
| Manage app-owned Marketplace event subscriptions | General App + `client_credentials` app-owned token |
| Receive events only | Webhook-only App |
| Receive events + call APIs | General App or S2S (with webhooks) |

## Resources

- **App types docs**: https://developers.zoom.us/docs/integrations/
- **Marketplace**: https://marketplace.zoom.us/
