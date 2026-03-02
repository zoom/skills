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

Both transports expose the same tools. Streamable HTTP is preferred because it supports
full bidirectional streaming over a single connection.

> **[CONTRIBUTOR NEEDED]** Confirm the total tool count. Originally documented as 12;
> Zoom Doc creation may be an additional tool. Run `zoom-mcp:list_available_tools` for
> the authoritative list.

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

## Agentic Retrieval API

The `search_meetings` tool does not perform a simple metadata filter. It uses AI Companion's
**Agentic Retrieval API** — a semantic search capability that queries across the actual
content of your meetings: what was said, AI-generated summaries, attached documents,
whiteboards, and recordings.

**What this enables:**
- Search by what was discussed ("meetings where we talked about pricing") not just topic name
- Search a time period and get a digest of all meeting content
- Results include the AI summary inline — the summary text is returned in the response,
  no separate fetch needed for most use cases
- Results include Zoom Doc URLs when documents were attached to the meeting

**Two result types:**
- **Recap meeting** — meetings with AI Companion data; includes inline summary, docs, recording reference, whiteboards
- **View recording** — meetings with cloud recordings; full recording details

**Why AI Companion must be enabled:** The Agentic Retrieval API depends on AI Companion
having processed the meeting. If AI Companion (Smart Recording, Meeting Summary) is not
enabled, meetings are not indexed and search returns no content.

> **[CONTRIBUTOR NEEDED]** Is there public documentation for the Agentic Retrieval API
> on https://developers.zoom.us/docs/? If so, link it here. Source: Zoom Developer
> Platform / AI Companion team.

## How ChatGPT Connects

The Zoom for ChatGPT app (available in ChatGPT's app connector directory) uses the same
`mcp-us.zoom.us` server infrastructure. Users authorize via Zoom Marketplace OAuth. The
app's core capabilities — searching meeting content, retrieving AI summaries, accessing
Zoom Docs, creating Zoom Docs — are all powered by the same MCP tools documented in
this skill.

**Workflows in this skill map directly to what the ChatGPT app does**, making it the
best available reference for real-world usage patterns.

## Tool Discovery

Two utility tools allow Claude to inspect the server dynamically:

- `zoom-mcp:list_available_tools` — returns the full tool list with descriptions
- `zoom-mcp:get_tool_details` — returns the complete parameter schema for any tool by name

Use `get_tool_details` when you need the exact parameter format for a less-common field.

## JSON-RPC Error Codes

The Zoom MCP Server returns standard MCP error codes at the JSON-RPC layer before Zoom
API errors are surfaced. See [references/error-codes.md](../references/error-codes.md) for
a full reference.
