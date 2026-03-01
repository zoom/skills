# OAuth Setup — Zoom MCP Server

## Overview

The Zoom MCP Server uses **User-managed OAuth** (not Server-to-Server). Each user authorizes
with their own Zoom account, and the resulting Bearer token is passed with every MCP request.
This means the server acts on behalf of the authorizing user — it can only access meetings,
recordings, and data that user has access to.

---

## Step 1: Create a User-Managed OAuth App

1. Go to [marketplace.zoom.us](https://marketplace.zoom.us) → **Develop** → **Build App**
2. Select **OAuth** as the app type (User-managed, not Server-to-Server)
3. Set a **Redirect URL** (can be `http://localhost` for personal use)
4. Note your **Client ID** and **Client Secret**

---

## Step 2: Configure Required Scopes

Add the following scopes to your app:

| Scope | Required for |
|-------|-------------|
| `meeting:read` | `search_meetings`, `list_meetings`, `get_meeting` |
| `meeting:write` | `create_meeting`, `update_meeting`, `delete_meeting` |
| `recording:read` | `list_recordings`, `get_recording` |
| `recording:write` | `delete_recording` |

Save the app after adding scopes.

---

## Step 3: Authorize and Get a Bearer Token

Complete the OAuth authorization flow to get an access token for your account:

1. Construct the authorization URL:
   ```
   https://zoom.us/oauth/authorize?response_type=code&client_id=YOUR_CLIENT_ID&redirect_uri=YOUR_REDIRECT_URI
   ```
2. Open in browser → sign in → click **Allow**
3. Copy the `code` from the redirect URL
4. Exchange for tokens:
   ```bash
   curl -X POST https://zoom.us/oauth/token \
     -u "CLIENT_ID:CLIENT_SECRET" \
     -d "grant_type=authorization_code&code=CODE&redirect_uri=REDIRECT_URI"
   ```
5. Copy the `access_token` from the response

---

## Step 4: Enable AI Companion (Required for Transcripts)

> **Critical prerequisite**: Transcript and meeting summary data is only available via the
> MCP server if AI Companion is enabled on your account.

In the Zoom web portal:
1. Go to **Admin → Account Management → Account Settings → AI Companion**
2. Enable **Smart Recording** (generates transcript files in cloud recordings)
3. Enable **Meeting Summary** (generates summaries)

Without these enabled, `get_recording` will return a `recording_files` array without VTT
or TRANSCRIPT entries — transcripts simply won't appear.

---

## Step 5: Add to Claude Code

```bash
claude mcp add --transport http zoom-mcp \
  https://mcp-us.zoom.us/mcp/zoom/streamable \
  --header "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  --scope user
```

Verify the connection:
```
zoom-mcp:get_user_profile
```
This should return your Zoom profile. If it returns `-32001`, the token is invalid or missing.

---

## Token Lifecycle

| Property | Details |
|----------|---------|
| **Expiry** | Access tokens expire after 1 hour |
| **Refresh** | Use the `refresh_token` to get a new `access_token` without re-authorizing |
| **Re-add** | When a token expires, re-run `claude mcp add` with the updated token |

**Refresh token exchange:**
```bash
curl -X POST https://zoom.us/oauth/token \
  -u "CLIENT_ID:CLIENT_SECRET" \
  -d "grant_type=refresh_token&refresh_token=YOUR_REFRESH_TOKEN"
```

---

## Environment Variables

Store credentials securely rather than hard-coding them:

```bash
ZOOM_CLIENT_ID=your_client_id
ZOOM_CLIENT_SECRET=your_client_secret
ZOOM_OAUTH_TOKEN=your_access_token
ZOOM_REFRESH_TOKEN=your_refresh_token
```

See also: [zoom-oauth skill](../../oauth/SKILL.md) for deeper OAuth implementation patterns
across all Zoom app types.
