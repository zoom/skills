---
name: zoom-mcp/contact-center-admin
description: Zoom Contact Center User Administration MCP guidance for users, queues, roles, skills, templates, routing profiles, inboxes, and phone numbers.
triggers:
  - "zoom contact center user administration mcp"
  - "zoom contact center users mcp"
  - "zoom contact center queues mcp"
  - "zoom contact center skills mcp"
---

# Zoom Contact Center User Administration MCP

Use this account-level server to administer Contact Center users and their relationships with
queues, supervisors, roles, skills, templates, routing profiles, inboxes, system statuses, and
phone numbers.

## Endpoint

`https://mcp.zoom.us/mcp/i_4E4145455B2bd9FE/streamable`

This is the exact instance-specific endpoint published in Zoom's catalog. Do not replace it
with a guessed generic slug. The repo MCP bundle registers it as
`zoom-contact-center-admin-mcp` in [../../../.mcp.json](../../../.mcp.json).

## Authentication and setup

This is account-level administration. Create an S2S Marketplace app with the required
Contact Center and number-management `:admin` scopes, mint a client-credentials MCP token,
then initialize and discover tools. Do not substitute the General Contact Center app template
or a user-level token. See [references/tools.md](references/tools.md) and the
[Marketplace S2S template](../../rest-api/assets/marketplace-app-creation-template-for-s2s-contact-center-user-administration-mcp.json).

Confirm destructive targets before delete or unassign calls, and use batch writes only after
validating the target account and input set.

## Official source

https://developers.zoom.us/docs/mcp/zoom-contact-center-user-administration-mcp-server/
