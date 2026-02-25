---
name: zoom-skills
description: |
  Zoom Developer Platform skills organized by product. Each folder contains product-specific 
  documentation and use cases. The general folder contains generic, cross-product skills 
  and references.
---

# Zoom Developer Platform Skills

This repository contains skills for building with Zoom APIs and SDKs, focused on web development.

## Repo Notes

- [tools/zoom-crawler/README.md](tools/zoom-crawler/README.md) - crawler usage for ingesting Zoom docs and API references into markdown
- [general/references/sdk-upgrade-workflow.md](general/references/sdk-upgrade-workflow.md) - reusable changelog + RSS version-by-version SDK upgrade workflow

## Folder Structure

| Folder | Description |
|--------|-------------|
| **[zoom-general](general/SKILL.md)** | **Hub** - Cross-product use cases and shared references |
| **[zoom-rest-api](rest-api/SKILL.md)** | REST API calls (600+ endpoints) |
| **[zoom-webhooks](webhooks/SKILL.md)** | Real-time event notifications (HTTP) |
| **[zoom-websockets](websockets/SKILL.md)** | Real-time event notifications (WebSocket) |
| **[zoom-meeting-sdk](meeting-sdk/SKILL.md)** | Embed Zoom meetings in your app (Web, React Native, Electron, Linux) |
| **[zoom-video-sdk](video-sdk/SKILL.md)** | Custom video experiences (Web, React Native, Flutter, Android, iOS, macOS, Unity, Linux) |
| **[zoom-apps-sdk](zoom-apps-sdk/SKILL.md)** | Apps that run inside the Zoom client |
| **[zoom-rtms](rtms/SKILL.md)** | Real-Time Media Streams (live audio/video/transcripts) |
| **[zoom-team-chat](team-chat/SKILL.md)** | Team Chat messaging and channels |
| **[contact-center](contact-center/SKILL.md)** | Contact Center apps, web embeds, and native mobile SDK integrations |
| **[phone](phone/SKILL.md)** | Zoom Phone APIs, Smart Embed, URI schemes, webhooks, and call-history migration patterns |
| **[rivet-sdk](rivet-sdk/SKILL.md)** | Rivet JavaScript server framework for OAuth, webhook events, and typed API endpoint wrappers |
| **[probe-sdk](probe-sdk/SKILL.md)** | Browser/device/network readiness diagnostics before Meeting SDK or Video SDK join flows |
| **[zoom-ui-toolkit](ui-toolkit/SKILL.md)** | Pre-built React components for Video SDK |
| **[zoom-cobrowse-sdk](cobrowse-sdk/SKILL.md)** | Collaborative browsing for support |
| **[zoom-oauth](oauth/SKILL.md)** | OAuth authentication flows (all 4 grant types) |

## How to Use

1. **Product-specific task?** → Go to that product's folder (e.g., `meeting-sdk/`)
2. **Cross-product or general task?** → Check [zoom-general](general/SKILL.md) for generic use cases
3. **Each folder has a `SKILL.md`** → Start there for that product's overview

## Skill Discovery (Integrated)

This repo uses progressive discovery:
- Agent reads `SKILL.md` frontmatter first (`name`, `description`, optional `triggers`).
- Agent routes by intent using `general/SKILL.md` "Choose Your Path" plus keyword/trigger matching.
- Agent loads full skill content only after route selection.

To improve routing quality in any skill:
- Keep `description` specific and keyword-rich (include "use when...").
- Add practical `triggers` for common user phrasing.
- Avoid overlapping triggers between Meeting SDK, Video SDK, and REST API intents.

## general (Hub)

The `general` folder contains:

- **Cross-product use cases** - Tasks that span multiple Zoom products
- **Generic references** - Authentication, app types, scopes
- **Shared documentation** - SDK maintenance, troubleshooting, marketplace publishing

Use `general` when:
- You're starting a new Zoom integration and need to choose the right approach
- Your use case involves multiple Zoom products
- You need general platform documentation (auth, scopes, etc.)

## Quick Reference

| I want to... | Use this skill |
|--------------|----------------|
| Make API calls (create meetings, manage users) | **zoom-rest-api** |
| Receive event notifications (HTTP push) | **webhooks** |
| Receive event notifications (WebSocket) | **zoom-websockets** |
| Embed Zoom meetings in my app | **zoom-meeting-sdk** |
| Build custom video experiences | **zoom-video-sdk** |
| Build an app inside Zoom client | **zoom-apps-sdk** |
| Access live audio/video/transcripts | **rtms** |
| Build Team Chat integrations | **zoom-team-chat** |
| Build Contact Center integrations (apps/web/mobile) | **contact-center** |
| Build Zoom Phone integrations (Smart Embed, APIs, webhooks, URI launch) | **phone** |
| Build Rivet-based server integrations (optional) | **rivet-sdk** |
| Run browser/device/network preflight diagnostics before join | **probe-sdk** |
| Use pre-built video UI components | **zoom-ui-toolkit** |
| Enable co-browsing for support | **zoom-cobrowse-sdk** |
| Implement OAuth authentication | **oauth** |
| General/cross-product guidance | **general** |

## Resources

- **Official docs**: https://developers.zoom.us/
- **Marketplace**: https://marketplace.zoom.us/
- **Developer forum**: https://devforum.zoom.us/

## Operations

- [RUNBOOK.md](RUNBOOK.md) - 5-minute preflight and debugging checklist.
