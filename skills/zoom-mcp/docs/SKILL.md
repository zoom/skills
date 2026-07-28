---
name: zoom-mcp/docs
description: |
  Legacy compatibility route for requests that call the current Zoom Canvas MCP server
  "Docs MCP". Route new work to zoom-mcp/canvas and use the Canvas endpoint, scopes, and tools.
triggers:
  - "zoom docs mcp"
  - "create zoom doc mcp"
  - "get zoom doc content mcp"
  - "create_file_with_content"
---

# Legacy Zoom Docs MCP Alias

Zoom's current official server is **Zoom Canvas MCP Server**, not a separate Docs MCP server.
Use [../canvas/SKILL.md](../canvas/SKILL.md) for new implementations. This file remains only
so older prompts and skill references containing "Docs MCP" continue to route correctly.

## Current Route

```text
https://mcp.zoom.us/mcp/canvas/streamable
```

See the [Canvas tool catalog](../canvas/references/tools.md) for current names and scopes.

## Official Source

https://developers.zoom.us/docs/mcp/zoom-canvas-mcp-server/
