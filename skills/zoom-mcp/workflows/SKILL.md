---
name: zoom-mcp/workflows
description: Zoom Workflows MCP guidance for inspecting, deploying, running, monitoring, and resuming user-owned Workflow Builder automations.
triggers:
  - "zoom workflows mcp"
  - "workflow builder mcp"
  - "deploy zoom workflow"
  - "monitor workflow execution"
---

# Zoom Workflows MCP

Use the dedicated Zoom Workflows MCP server for lifecycle operations on user-owned Workflow
Builder automations: inspect node contracts, deploy or undeploy workflows, run them, and monitor
or resume executions.

Registry identity: `io.github.zoom/zoom-workflows`, version 1.0.0 in the 2026-11-17 snapshot.

## Endpoint

```text
https://mcp.zoom.us/mcp/workflow/streamable
```

The repo MCP bundle registers it as `zoom-workflows-mcp` in [../../../.mcp.json](../../../.mcp.json).

## Setup

Complete authorization through the parent workflow in [../SKILL.md](../SKILL.md). Workflow
objects are user-owned, so use a user token with only the scopes required by `tools/list`; do
not substitute an account-level token unless the live tool contract requires it.

Representative capabilities from registry metadata:

- List user-owned workflows and read saved definitions.
- Inspect node contracts before deployment.
- Deploy or undeploy agentic workflows.
- Run asynchronously, check status, and resume paused executions.

Always inspect the node contract before deploying; confirm the trigger, inputs, outputs, and
side effects with the workflow owner.

## Official Source

https://developers.zoom.us/docs/mcp/zoom-workflows-mcp-server/
