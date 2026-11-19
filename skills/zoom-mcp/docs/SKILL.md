---
name: zoom-mcp/docs
description: |
  Zoom Docs MCP guidance and legacy router. Use the dedicated Docs endpoint for Zoom Docs and
  notes; route older "Docs MCP" requests that expect Canvas tools to zoom-mcp/canvas.
triggers:
  - "zoom docs mcp"
  - "create zoom doc mcp"
  - "get zoom doc content mcp"
  - "create_file_with_content"
---

# Zoom Docs MCP

Zoom has a dedicated Docs MCP server and a separate Canvas MCP server. Use Docs for creating or
reading Zoom Docs and notes as Markdown. Use Canvas for file, block, collaborator, and content
operations.

## Dedicated Docs Server

Registry identity: `io.github.zoom/zoom-docs`, version 1.0.2 in the 2026-11-17 snapshot.

## Current Route

```text
https://mcp.zoom.us/mcp/docs/streamable
```

The repo MCP bundle registers it as `zoom-docs-mcp` in [../../../.mcp.json](../../../.mcp.json).
Call `tools/list` for the current names and scopes.

## Legacy Canvas Alias

Some existing prompts call the Canvas server "Docs MCP." For those flows, use
[../canvas/SKILL.md](../canvas/SKILL.md) and its endpoint:

```text
https://mcp.zoom.us/mcp/canvas/streamable
```

## Official Source

https://developers.zoom.us/docs/mcp/zoom-docs-mcp-server/
