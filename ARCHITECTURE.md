# Skills Repository Architecture

This document describes the structure and routing model of the Zoom skills repository.

## Overview

The repository uses a hub-and-spoke model:

- `general` is the hub for routing and cross-product guidance.
- Product folders are spokes with product-specific `SKILL.md` files and deeper references/examples.
- `SKILL.md` is the canonical entry file per skill directory.

See also: [SKILL.md](SKILL.md)

## Hub-and-Spoke Routing

```text
User request
  -> general/SKILL.md (intent routing)
  -> product SKILL.md (implementation path)
  -> references/examples/troubleshooting (deep detail)
```

Common route outcomes:

- API management flows -> `rest-api`
- AI-driven meeting search, meeting-asset retrieval, recording-resource retrieval, and Zoom Docs workflows -> `zoom-mcp`
- Whiteboard-specific MCP workflows -> `zoom-mcp/whiteboard`
- Enterprise AI systems requiring stable API core + AI tool layer -> `rest-api` + `zoom-mcp`
- Event notifications -> `webhooks` or `websockets`
- Embedded meetings -> `meeting-sdk`
- Custom video sessions -> `video-sdk`
- In-client apps -> `zoom-apps-sdk`
- Live media streams -> `rtms`
- Contact Center -> `contact-center`
- Virtual Agent web/mobile chat and KB sync -> `virtual-agent`
- Phone integrations -> `phone`
- Rivet server integrations -> `rivet-sdk`
- Pre-join diagnostics/readiness -> `probe-sdk`

## Directory Structure

```text
skills/
├── SKILL.md
├── ARCHITECTURE.md
├── CONTRIBUTING.md
├── general/
├── rest-api/
├── webhooks/
├── websockets/
├── meeting-sdk/
├── video-sdk/
├── zoom-apps-sdk/
├── rtms/
├── team-chat/
├── contact-center/
├── virtual-agent/
├── phone/
├── rivet-sdk/
├── probe-sdk/
├── ui-toolkit/
├── cobrowse-sdk/
├── oauth/
├── zoom-mcp/
│   └── whiteboard/
└── tools/
```

Platform-heavy spokes (examples):

- `meeting-sdk`: web, android, ios, macos, unreal, electron, react-native, linux, windows
- `video-sdk`: web, android, ios, macos, unity, react-native, flutter, linux, windows
- `contact-center`: common + android + ios + web
- `virtual-agent`: common + android + ios + web (campaign/chat + WebView bridges)

## Design Principles

1. One entry file per skill directory
- Use `SKILL.md` as the canonical entrypoint.
- Keep `SKILL.md` focused on routing, lifecycle, and key links.

2. Progressive disclosure
- Keep high-level guidance in `SKILL.md`.
- Put depth in `references/`, `examples/`, `concepts/`, and `troubleshooting/`.

3. Link integrity
- Every `.md` should be reachable from a parent `SKILL.md` or index-like parent doc.
- Prefer relative repository links; avoid machine-local absolute paths.

4. Cross-skill chaining
- Put multi-product workflows in `general/use-cases/`.
- Link chained skills explicitly from those use cases.

## Product Skill Summary

| Skill | Purpose |
|---|---|
| `general` | Hub routing and cross-product patterns |
| `rest-api` | Zoom REST API operations |
| `webhooks` | HTTP event ingestion |
| `websockets` | Persistent event streams |
| `meeting-sdk` | Embed Zoom meetings in apps |
| `video-sdk` | Build custom video session experiences |
| `zoom-apps-sdk` | Build apps inside Zoom client surfaces |
| `rtms` | Real-time media streams |
| `team-chat` | Team Chat integrations |
| `contact-center` | Contact Center app/web/mobile integrations |
| `virtual-agent` | Virtual Agent web/mobile chat embeds and knowledge-base sync patterns |
| `phone` | Phone APIs, Smart Embed, URI/webhook flows |
| `rivet-sdk` | Rivet JavaScript server framework for auth, events, and API wrappers |
| `probe-sdk` | Browser/device/network readiness diagnostics before meeting/video join |
| `ui-toolkit` | Prebuilt Video SDK UI components |
| `cobrowse-sdk` | Collaborative browsing support flows |
| `oauth` | Auth flows and token models |
| `zoom-mcp` | Zoom-hosted MCP server workflows for semantic meeting search, meeting assets, recording resources, and Zoom Docs creation |
| `zoom-mcp/whiteboard` | Whiteboard MCP endpoints, scopes, auth findings, ID mapping, and live Whiteboard tool workflows |

## Maintenance Notes

- Use `tools/md-audit/md_audit.py` to validate links/orphans after large doc changes.
- Keep naming/folder conventions aligned with root `SKILL.md`.
- Update architecture references when adding/removing product folders.
