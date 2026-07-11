---
name: zoom-mcp/revenue-accelerator
description: |
  Zoom Revenue Accelerator MCP server guidance for conversation transcripts and analysis,
  scorecards, deals, customer accounts/contacts, indicators, teams, and users. Use when an AI
  agent needs read-oriented ZRA sales intelligence through the dedicated MCP endpoint.
triggers:
  - "zoom revenue accelerator mcp"
  - "zra mcp"
  - "conversation analysis mcp"
  - "deal analysis mcp"
  - "sales conversation transcript zoom"
---

# Zoom Revenue Accelerator MCP

Use this read-oriented MCP surface for licensed Zoom Revenue Accelerator data. Do not route
ordinary meeting transcripts here unless the data belongs to a ZRA conversation.

## Endpoint

`https://mcp.zoom.us/mcp/revenue_accelerator/streamable`

The repo MCP bundle registers it as `zoom-revenue-accelerator-mcp` in
[../../../.mcp.json](../../../.mcp.json).

## Tools

See [references/tools.md](references/tools.md) for the current 15-tool catalog and scopes.

## Guardrails

- Require the relevant ZRA license, data access, and CRM configuration.
- Use search tools to resolve conversation, deal, account, contact, team, and user IDs.
- Do not infer that ordinary cloud recordings are automatically ZRA conversations.
- Prefer deterministic REST APIs when the workflow needs bulk extraction, strict retries, or
  non-agent automation.

## Chaining

- Marketplace app creation: [Revenue Accelerator MCP template](../../rest-api/assets/marketplace-apps/zoom-mcp-revenue-accelerator.json)
  via [Marketplace app management](../../rest-api/references/marketplace-apps.md)
- Token acquisition: create the app first, then use [OAuth and PKCE](../../oauth/SKILL.md) to
  authorize the licensed user and mint the bearer token supplied to this MCP endpoint
- Parent MCP routing: [../SKILL.md](../SKILL.md)
- Revenue Accelerator REST inventory: [../../rest-api/references/revenue-accelerator.md](../../rest-api/references/revenue-accelerator.md)

## Official Source

https://developers.zoom.us/docs/mcp/zoom-revenue-accelerator-mcp-server/
