# OAuth Setup — Zoom MCP Server

## Overview

The primary documented path for Zoom MCP is **user OAuth**. Each user authorizes with their
own Zoom account, and the resulting bearer token is sent with MCP requests.

A **Server-to-Server OAuth** token can initialize against the MCP gateway and complete
`tools/list`. That means S2S is not outright blocked at the transport layer. Do not assume
tool parity, though: actual execution is still scope-gated and must be verified per token
type and MCP server.

## Step 1: Create a User OAuth App

1. Go to [marketplace.zoom.us](https://marketplace.zoom.us) → **Develop** → **Build App**.
2. Select **OAuth** for the per-user MCP path.
3. Set a redirect URL for your client or local test environment.
4. Note the client ID and client secret.

## Step 2: Configure Zoom MCP Scopes

Add the MCP-specific granular scopes required by the tools you want to use.

| Scope | Needed for |
|-------|------------|
| `meeting:read:search` | `search_meetings` |
| `meeting:read:assets` | `get_meeting_assets` |
| `cloud_recording:read:list_user_recordings` | `recordings_list` |
| `cloud_recording:read:content` | `get_recording_resource` |
| `docs:write:import` | `create_new_file_with_markdown` |

Whiteboard MCP uses a separate scope set. See [../whiteboard/SKILL.md](../whiteboard/SKILL.md).

## Step 3: Authorize and Exchange for Tokens

1. Construct the authorization URL:
   ```
   https://zoom.us/oauth/authorize?response_type=code&client_id=YOUR_CLIENT_ID&redirect_uri=YOUR_REDIRECT_URI
   ```
2. Sign in and approve the app.
3. Copy the `code` from the redirect URL.
4. Exchange the code for tokens:
   ```bash
   curl -X POST https://zoom.us/oauth/token \
     -u "CLIENT_ID:CLIENT_SECRET" \
     -d "grant_type=authorization_code&code=CODE&redirect_uri=REDIRECT_URI"
   ```
5. Store the returned `access_token` and `refresh_token`.

## Step 4: Enable AI Companion Features

Smart Recording and Meeting Summary are feature prerequisites for useful semantic meeting
search, meeting assets, and transcript-rich recording content.

In the Zoom web portal:
1. Go to **Admin → Account Management → Account Settings → AI Companion**.
2. Enable **Smart Recording**.
3. Enable **Meeting Summary**.

Important:
- these settings do **not** replace the OAuth scopes above
- they affect whether useful recap and transcript content exists for MCP retrieval

## Step 5: Add the MCP Server to Your Client

Claude Code example:

```bash
claude mcp add --transport http zoom-mcp \
  https://mcp-us.zoom.us/mcp/zoom/streamable \
  --header "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  --scope user
```

Verification:
- confirm the client can see `recordings_list`, `search_meetings`, `get_meeting_assets`,
  `get_recording_resource`, and `create_new_file_with_markdown`
- if your client exposes protocol inspection, use `tools/list` as the authority for the live catalog
- run a simple tool such as `recordings_list` to verify the token has the correct MCP scopes

## Token Lifecycle

| Property | Details |
|----------|---------|
| Access token expiry | About 1 hour |
| Refresh flow | Exchange `refresh_token` for a new `access_token` |
| Client update | Refresh the bearer token used by the MCP client registration |

Refresh exchange:

```bash
curl -X POST https://zoom.us/oauth/token \
  -u "CLIENT_ID:CLIENT_SECRET" \
  -d "grant_type=refresh_token&refresh_token=YOUR_REFRESH_TOKEN"
```

## Optional: Test S2S Separately

If you want to test S2S against the MCP gateway:
- verify the token can initialize and complete `tools/list`
- verify each required tool individually
- do not assume that S2S supports the same tool execution path as user OAuth

## Environment Variables

```bash
ZOOM_CLIENT_ID=your_client_id
ZOOM_CLIENT_SECRET=your_client_secret
ZOOM_OAUTH_TOKEN=your_access_token
ZOOM_REFRESH_TOKEN=your_refresh_token
```

See also: [../../oauth/SKILL.md](../../oauth/SKILL.md)
