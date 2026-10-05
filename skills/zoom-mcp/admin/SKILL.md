---
name: zoom-mcp/admin
description: Zoom Admin MCP guidance for account settings, groups, users, and permissions. Use for account-level administration through the dedicated Admin MCP server.
triggers:
  - "zoom admin mcp"
  - "zoom account settings mcp"
  - "zoom users mcp"
  - "zoom groups mcp"
---

# Zoom Admin MCP

Use the account-level Admin MCP server for read access to Zoom users, groups, permissions,
account settings, and lock settings, plus account settings updates.

## Endpoint

`https://mcp.zoom.us/mcp/admin/streamable`

The repo MCP bundle registers it as `zoom-admin-mcp` in [../../../.mcp.json](../../../.mcp.json).

## Authentication and setup

This server is account-level and uses an S2S/OAuth-style MCP access token. Create an S2S
Marketplace app with only the required `:admin` or `:master` scopes, obtain its client
credentials, mint the client-credentials token, then initialize the MCP session. Do not use a
user-level General App token for account administration.

See [references/tools.md](references/tools.md) for the complete tool/scope inventory and
[Marketplace S2S template](../../rest-api/assets/marketplace-app-creation-template-for-s2s-mcp-admin.json).

## Safety

Treat settings updates as account-wide changes. Confirm the target setting and account before
calling `account_settings_update`; prefer reads first and record the response for audit.

## Official source

https://developers.zoom.us/docs/mcp/zoom-admin-mcp-server/
