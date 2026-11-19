---
name: zoom-mcp/phone
description: Zoom Phone MCP guidance for administering sites, users, calling plans, numbers, call queues, auto receptionists, shared lines, common areas, emergency addresses, monitoring, and push-to-talk.
triggers:
  - "zoom phone mcp"
  - "phone call queue mcp"
  - "zoom number assignment mcp"
  - "zoom phone admin mcp"
---

# Zoom Phone MCP

Use the dedicated Zoom Phone MCP server for Phone administration: sites, users, calling plans,
phone numbers, call queues, auto receptionists, shared line groups, common areas, monitoring
groups, emergency addresses, and push-to-talk.

Registry identity: `io.github.zoom/zoom-phone`, version 1.0.1 in the 2026-11-17 snapshot.

## Endpoint

```text
https://mcp.zoom.us/mcp/phone/streamable
```

The repo MCP bundle registers it as `zoom-phone-mcp` in [../../../.mcp.json](../../../.mcp.json).

## Setup

Follow the parent OAuth workflow in [../SKILL.md](../SKILL.md), request only the Phone scopes
required by `tools/list`, then call `initialize` and `tools/list` before automation. Registry
metadata documents administration examples but does not enumerate tool names; the live
`tools/list` response is authoritative.

Representative capabilities from registry metadata:

- Create and manage call queues, members, numbers, and handling settings.
- Assign numbers and calling plans to users and common areas.
- Discover and invoke supported Phone read and write operations.
- Review sites, auto receptionists, shared line groups, and emergency addresses.

For deterministic bulk changes, audit reports, or operations absent from `tools/list`, chain
[Zoom REST API](../../rest-api/SKILL.md).

## Official Source

https://developers.zoom.us/docs/mcp/zoom-phone-mcp-server/
