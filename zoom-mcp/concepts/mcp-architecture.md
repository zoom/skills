# MCP Architecture — Zoom MCP Server

## What is MCP?

Model Context Protocol (MCP) is an open protocol introduced by Anthropic that standardizes
how AI systems connect to external tools and data sources. It uses JSON-RPC 2.0 as its
messaging format — the AI sends a method call, the server returns a result.

The Zoom MCP Server implements MCP to expose Zoom's meeting management and recording
capabilities as callable tools.

## The Official Zoom MCP Server

Zoom operates an official, hosted MCP server at `mcp-us.zoom.us`:

| Transport | URL |
|-----------|-----|
| **Streamable HTTP** (recommended) | `https://mcp-us.zoom.us/mcp/zoom/streamable` |
| **SSE** (Server-Sent Events, fallback) | `https://mcp-us.zoom.us/mcp/zoom/sse` |

Both transports expose the same 12 tools. Streamable HTTP is preferred because it supports
full bidirectional streaming over a single connection.

A separate Whiteboard MCP server exists at `/mcp/whiteboard/streamable` — it requires a
Whiteboard instance ID and is not covered by this skill.

## How Claude Connects

When you run `claude mcp add`, Claude Code registers the server:

```bash
claude mcp add --transport http zoom-mcp \
  https://mcp-us.zoom.us/mcp/zoom/streamable \
  --header "Authorization: Bearer YOUR_ZOOM_OAUTH_TOKEN" \
  --scope user
```

At conversation start, Claude calls `tools/list` to discover available tools and their
parameter schemas. When you ask Claude to "search my meetings" or "get a transcript", Claude
selects the appropriate tool (`zoom-mcp:search_meetings`, `zoom-mcp:get_recording`), validates
parameters against the schema, and calls `tools/call`.

## How ChatGPT Connects

The Zoom for ChatGPT app (available in ChatGPT's app connector directory) uses the same
`mcp-us.zoom.us` server infrastructure via OpenAI's MCP connector support. Users authorize
via Zoom Marketplace OAuth, and the same 12 tools are exposed. The core use case in ChatGPT
is identical: search meetings → retrieve recordings → analyze transcripts.

This means: **workflows documented in this skill map directly to what the ChatGPT app does**,
which is the best available signal for what users actually need.

## Tool Discovery

Two utility tools allow Claude to inspect the server dynamically:

- `zoom-mcp:list_available_tools` — returns the full tool list with descriptions
- `zoom-mcp:get_tool_details` — returns the complete parameter schema for any tool by name

Use `get_tool_details` when you need the exact parameter format for a less-common field.

## JSON-RPC Error Codes

The Zoom MCP Server returns standard MCP error codes at the JSON-RPC layer before Zoom
API errors are surfaced. See [references/error-codes.md](../references/error-codes.md) for
a full reference.
