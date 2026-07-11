# OAuth Setup — Zoom MCP Server

## Overview

The primary documented path for Zoom MCP is **user OAuth**. Each user authorizes with their
own Zoom account, and the resulting bearer token is sent with MCP requests.

A **Server-to-Server OAuth** token can initialize against the MCP gateway and complete
`tools/list`. That means S2S is not outright blocked at the transport layer. Do not assume
tool parity, though: actual execution is still scope-gated and must be verified per token
type and MCP server.

## Required Skill Chain

Use this order for every Zoom-hosted MCP server:

1. **Marketplace app creation:** Route to
   [Marketplace app management](../../rest-api/references/marketplace-apps.md), choose the
   matching MCP template from the
   [Marketplace template selector](../../rest-api/references/marketplace-app-templates.md),
   and create a user-managed General App.
2. **OAuth and token acquisition:** Return here and use [zoom-oauth](../../oauth/SKILL.md) to
   configure the redirect URI, authorize the user, exchange the code, and store the access and
   refresh tokens.
3. **MCP connection:** Pass the access token as `Authorization: Bearer ...`, initialize the
   selected MCP endpoint, discover its current tools, and then invoke tools.

Do not start at the MCP endpoint without first establishing which Marketplace app owns the
credentials and whether its granted scopes cover the selected tools.

## Step 1: Create a User-Managed General App

1. Go to [marketplace.zoom.us](https://marketplace.zoom.us) → **Develop** → **Build App**.
2. Select a **General App** with user-managed OAuth for the per-user MCP path.
3. Start from the matching JSON template in
   [Marketplace app templates](../../rest-api/references/marketplace-app-templates.md).
4. Set a redirect URL for your client or local test environment.
5. Note the client ID and protect the client secret if the selected client type uses one.

## Step 2: Configure Zoom MCP Scopes

Add the MCP-specific granular scopes required by the tools you want to use.

| Scope | Needed for |
|-------|------------|
| `meeting:read:search` | `search_meetings` |
| `meeting:read:assets` | `get_meeting_assets` |
| `ai_companion:read:search` | `search_zoom` |
| `cloud_recording:read:list_user_recordings` | `recordings_list` |
| `cloud_recording:read:content` | `get_recording_resource` |
| `docs:write:import` | `create_new_file_with_markdown` |
| `docs:read:export` | `get_file_content` |
| `hub:write:content` | `hub_create_file_from_content` |
| `hub:read:content` | `hub_get_file_content` |

Dedicated product MCP servers use separate, least-privilege scope sets. See:
- [../whiteboard/SKILL.md](../whiteboard/SKILL.md)
- [../team-chat/SKILL.md](../team-chat/SKILL.md)
- [../meetings/SKILL.md](../meetings/SKILL.md)
- [../docs/SKILL.md](../docs/SKILL.md)
- [../tasks/SKILL.md](../tasks/SKILL.md)
- [../revenue-accelerator/SKILL.md](../revenue-accelerator/SKILL.md)

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
  https://mcp.zoom.us/mcp/zoom/streamable \
  --header "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  --scope user
```

Verification:
- confirm the client can see 9 default Zoom MCP tools: `search_meetings`,
  `create_new_file_with_markdown`, `search_zoom`, `get_meeting_assets`,
  `get_recording_resource`, `get_file_content`, `recordings_list`,
  `hub_create_file_from_content`, and `hub_get_file_content`
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
