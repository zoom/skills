# OAuth Scopes

OAuth scopes define what your app can access.

## Overview

Scopes are permissions requested during OAuth authorization. Request only the scopes you need.

## IMPORTANT: Scope Types

**Different current app types and General App scope levels have different scopes available:**

| App / scope mode | Scope Suffix | Access Level | Example |
|------------------|--------------|--------------|---------|
| **General App, user-level scoped** | Usually no `:admin` suffix | Current user's data only | `meeting:read` |
| **General App, admin/account-level scoped** | Commonly `:admin` | Account resources granted by admin authorization | `meeting:read:admin` |
| **Server-to-Server OAuth** | Commonly `:admin` | Own account automation, no user consent | `meeting:read:admin` |
| **General App, app-owned client credentials** | Endpoint-specific app scopes | App-owned capabilities, not user-owned data | `marketplace:write:event_subscription` |

### Key Differences

- **User-level General App scopes** (`meeting:read`): Access only the authenticated user's data.
- **Admin/account-level General App scopes** (`meeting:read:admin`): Access account resources granted by admin authorization.
- **S2S OAuth**: Uses account-level scopes without user login, intended for backend integrations on your own account.
- **App-owned General App scopes**: Use `grant_type=client_credentials` with the General App Client ID and Client Secret when the endpoint requires it.

### Choosing the Right Scope Type

| Use Case | OAuth Type | Scope Example |
|----------|------------|---------------|
| User manages their own meetings | General App, user-level scoped | `meeting:write` |
| Admin dashboard for all users | General App, admin/account-level scoped | `meeting:read:admin` |
| Backend automation (no user login) | Server-to-Server | `meeting:write:admin` |
| Bot that creates meetings for users | Server-to-Server | `meeting:write:admin` |
| App configures its own Marketplace event subscriptions | General App, app-owned client credentials | `marketplace:write:event_subscription` |

## Common Scopes

### Meetings

| User Scope | Admin Scope | Description |
|------------|-------------|-------------|
| `meeting:read` | `meeting:read:admin` | View meeting details |
| `meeting:write` | `meeting:write:admin` | Create, update, delete meetings |
| `meeting:master` | `meeting:master:admin` | Full meeting access |

### Users

| User Scope | Admin Scope | Description |
|------------|-------------|-------------|
| `user:read` | `user:read:admin` | View user profile |
| `user:write` | `user:write:admin` | Update user settings |
| `user:master` | `user:master:admin` | Full user access |

### Recordings

| User Scope | Admin Scope | Description |
|------------|-------------|-------------|
| `recording:read` | `recording:read:admin` | View/download recordings |
| `recording:write` | `recording:write:admin` | Delete recordings |
| `recording:master` | `recording:master:admin` | Full recording access |

### Webinars

| User Scope | Admin Scope | Description |
|------------|-------------|-------------|
| `webinar:read` | `webinar:read:admin` | View webinar details |
| `webinar:write` | `webinar:write:admin` | Create, update webinars |
| `webinar:master` | `webinar:master:admin` | Full webinar access |

### Reports

| User Scope | Admin Scope | Description |
|------------|-------------|-------------|
| `report:read` | `report:read:admin` | View reports and analytics |
| `report:master` | `report:master:admin` | Full report access |

## Scope Patterns

| Pattern | Meaning |
|---------|---------|
| `resource:read` | Read-only access (current user) |
| `resource:write` | Read and write access (current user) |
| `resource:master` | Full access including delete (current user) |
| `resource:read:admin` | Read-only access (all account users) |
| `resource:write:admin` | Read and write access (all account users) |
| `resource:master:admin` | Full access including delete (all account users) |

## Best Practices

1. **Request minimum scopes** - Only what you need
2. **Explain to users** - Why you need each scope
3. **Handle denied scopes** - Graceful fallback

## Resources

- **Scopes reference**: https://developers.zoom.us/docs/integrations/oauth-scopes/
