# Zoom Developer Platform Skills

Skills for building with Zoom SDKs, APIs, MCP servers, and integrations across web, mobile, desktop, and server environments.

Primary skill entrypoint: [skills/SKILL.md](skills/SKILL.md)

## Installation

### Claude Code Plugin Install

This repository is packaged as a Claude plugin:
- plugin manifest: [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json)
- marketplace manifest: [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json)
- bundled MCP servers: [`.mcp.json`](.mcp.json)

Install from GitHub using Claude Code:
```bash
/plugin marketplace add zoom/skills
/plugin install zoom-skills@zoom-skills
```

Or add the local checkout as a marketplace source:
```bash
/plugin marketplace add /absolute/path/to/zoom-skills
/plugin install zoom-skills@zoom-skills
```

### Native Agent Skills Installs

These editors document native `SKILL.md` support or compatible skill directories:

| Agent | Install Model | Location | Docs |
|-------|---------------|----------|------|
| Cline | Native skills | `~/.cline/skills/` or `.cline/skills/` | [skills](https://docs.cline.bot/customization/skills) |
| BLACKBOX AI | Native skills | `.blackbox/skills/` | [skills](https://docs.blackbox.ai/features/blackbox-cli/skills) |

Clone once:

```bash
git clone https://github.com/zoom/skills.git ~/zoom-skills
```

Cline raw skill install:

```bash
mkdir -p ~/.cline/skills
find ~/zoom-skills/skills -mindepth 1 -maxdepth 1 -type d -exec cp -r {} ~/.cline/skills/ \;
```

BLACKBOX AI project install:

```bash
mkdir -p .blackbox/skills
find ~/zoom-skills/skills -mindepth 1 -maxdepth 1 -type d -exec cp -r {} .blackbox/skills/ \;
```

If you previously installed this repo under the older `agent-skills` name, remove the old folder so your agent does not load duplicates.

### Agents Without Native SKILL.md Installs

These editors do not expose a documented first-class `SKILL.md` install flow comparable to Cline or BLACKBOX AI. Use the nearest equivalent:

| Tool | Recommended Integration | Docs |
|------|-------------------------|------|
| Roo Code | Port the relevant skill into Custom Modes and add MCP servers separately | [docs](https://docs.roocode.com/) |
| Kilo Code | Port the relevant skill into Custom Modes / Custom Rules and add MCP servers separately | [docs](https://kilocode.ai/docs) |

Recommended starting point for Roo Code or Kilo Code:

1. Start from [skills/general/SKILL.md](skills/general/SKILL.md) for routing and product selection.
2. Copy the target product skill into your mode/rule prompt, for example:
   - [skills/rest-api/SKILL.md](skills/rest-api/SKILL.md)
   - [skills/meeting-sdk/SKILL.md](skills/meeting-sdk/SKILL.md)
   - [skills/video-sdk/SKILL.md](skills/video-sdk/SKILL.md)
   - [skills/zoom-mcp/SKILL.md](skills/zoom-mcp/SKILL.md)
3. Add the Zoom MCP servers separately from [`.mcp.json`](.mcp.json) if your editor supports MCP.

### Cursor

Cursor packaging metadata is included for marketplace-style consumers:
- plugin manifest: [`.cursor-plugin/plugin.json`](.cursor-plugin/plugin.json)
- marketplace manifest: [`.cursor-plugin/marketplace.json`](.cursor-plugin/marketplace.json)
- repo rules: [`.cursor/rules/`](.cursor/rules/)

### Context7

Skills are automatically discovered when the repository is indexed by Context7.

## Getting Started

### 1. Ask Claude about Zoom development

Once installed, simply ask questions about Zoom development:

```
How do I create a meeting using the Zoom API?
```

```
How do I build a meeting bot that joins and records?
```

```
What's the difference between Meeting SDK and Video SDK?
```

### 2. The agent loads the right skill automatically

The **general** skill acts as a router and directs to the appropriate specialized skill:

| Your Question | Skill Loaded |
|---------------|--------------|
| "Create a meeting via API" | zoom-rest-api |
| "Embed Zoom in my React app" | zoom-meeting-sdk |
| "Build custom video UI" | zoom-video-sdk |
| "Handle webhook events" | webhooks |
| "Build a meeting bot" | zoom-meeting-sdk + rtms |
| "Set up OAuth authentication" | oauth |
| "Build AI-agent meeting search tools" | zoom-mcp |

### 3. Skills chain automatically

When your task requires multiple skills, the agent loads them as needed. For example, "build a meeting bot" loads:
- **zoom-meeting-sdk** (for joining meetings)
- **rtms** (for real-time audio/video/transcript access)
- **zoom-rest-api** (for creating meetings)

## Skills

| Skill | Description |
|-------|-------------|
| [zoom-general](skills/general/) | **Hub** - Core concepts, authentication, use cases, routing |
| [zoom-rest-api](skills/rest-api/) | 600+ REST API endpoints, rate limits, pagination |
| [zoom-webhooks](skills/webhooks/) | Real-time event notifications |
| [zoom-websockets](skills/websockets/) | Real-time WebSocket event connections |
| [zoom-meeting-sdk](skills/meeting-sdk/) | Embed Zoom meetings (Web, React Native, Electron, Linux headless bots) |
| [zoom-video-sdk](skills/video-sdk/) | Custom video experiences (Web, React Native, Flutter, Linux headless bots) |
| [zoom-apps-sdk](skills/zoom-apps-sdk/) | Apps that run inside Zoom client |
| [scribe](skills/scribe/) | AI Services Scribe for uploaded-file and batch archive transcription |
| [zoom-rtms](skills/rtms/) | Real-time Media Streams (live audio/video/transcripts) |
| [zoom-team-chat](skills/team-chat/) | Team Chat APIs and integrations |
| [virtual-agent](skills/virtual-agent/) | Virtual Agent web embeds, Android/iOS wrappers, and KB sync workflows |
| [contact-center](skills/contact-center/) | Contact Center apps, web embeds, and Android/iOS native SDKs |
| [phone](skills/phone/) | Zoom Phone APIs, Smart Embed, URI schemes, and webhook patterns |
| [rivet-sdk](skills/rivet-sdk/) | Rivet JavaScript server framework for auth, webhooks, and typed endpoint wrappers |
| [probe-sdk](skills/probe-sdk/) | Browser/device/network readiness diagnostics before Meeting SDK or Video SDK joins |
| [zoom-ui-toolkit](skills/ui-toolkit/) | Pre-built UI components for Video SDK |
| [zoom-cobrowse-sdk](skills/cobrowse-sdk/) | Collaborative browsing for support |
| [zoom-oauth](skills/oauth/) | OAuth authentication (all 4 grant types) |
| [zoom-mcp](skills/zoom-mcp/) | Zoom-hosted MCP server workflows for AI-agent tooling, meeting summaries, and transcript retrieval |

## Common Use Cases

| Use Case | Skills Needed |
|----------|---------------|
| Schedule meetings programmatically | zoom-rest-api |
| Build meeting bots (AI/transcription) | zoom-meeting-sdk + rtms |
| Embed meetings in your app | zoom-meeting-sdk |
| Custom video experiences | zoom-video-sdk |
| Auto-download recordings to S3/GCS | webhooks + zoom-rest-api |
| Real-time AI processing | rtms |
| Batch or on-demand media transcription | scribe |
| In-meeting collaborative apps | zoom-apps-sdk |
| Team Chat integrations | zoom-team-chat |
| Virtual Agent campaign/chat flows (web + mobile wrappers) | virtual-agent + contact-center |
| Contact Center app/web/mobile integrations | contact-center |
| Rivet-based event-driven API backend | rivet-sdk + oauth + zoom-rest-api |
| Pre-join/browser readiness diagnostics | probe-sdk + meeting-sdk or video-sdk |
| Low-latency event notifications | zoom-websockets |
| OAuth authentication setup | oauth |
| AI-driven tool workflows over Zoom data | zoom-mcp |
| Enterprise AI architecture (API core + AI tool layer) | zoom-rest-api + zoom-mcp |

## Architecture

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full hub-and-spoke structure diagram.

```
zoom-general (HUB)
       │
       ├── zoom-rest-api
       ├── zoom-webhooks
       ├── zoom-websockets
       ├── zoom-meeting-sdk
       ├── zoom-video-sdk
       ├── zoom-apps-sdk
       ├── scribe
       ├── zoom-rtms
       ├── zoom-team-chat
       ├── virtual-agent
       ├── contact-center
       ├── phone
       ├── rivet-sdk
       ├── probe-sdk
       ├── zoom-ui-toolkit
       ├── zoom-cobrowse-sdk
       ├── zoom-oauth
       └── zoom-mcp
```

## Directory Structure

```text
repo/
├── .claude-plugin/           # Claude plugin and marketplace manifests
├── .cursor/                  # Cursor rules and repo guidance
├── .cursor-plugin/           # Cursor plugin and marketplace manifests
├── .mcp.json                 # Bundled Zoom MCP server endpoints
├── README.md                 # Packaging and installation overview
├── ARCHITECTURE.md           # Full architecture diagram
├── CONTRIBUTING.md           # Contribution guidance
├── RUNBOOK.md                # Repo-level maintenance notes
├── skills/                   # Installable skill tree
│   ├── SKILL.md              # Skill bundle entrypoint
│   ├── general/              # HUB (entry point)
│   │   ├── SKILL.md
│   │   ├── references/       # Cross-cutting docs
│   │   └── use-cases/        # Multi-skill scenarios
│   ├── rest-api/
│   ├── webhooks/
│   ├── websockets/
│   ├── meeting-sdk/
│   ├── video-sdk/
│   ├── zoom-apps-sdk/
│   ├── rtms/
│   ├── team-chat/
│   ├── contact-center/
│   ├── virtual-agent/
│   ├── phone/
│   ├── rivet-sdk/
│   ├── probe-sdk/
│   ├── ui-toolkit/
│   ├── cobrowse-sdk/
│   ├── oauth/
│   └── zoom-mcp/
```

## Resources

- [Zoom Developer Platform](https://developers.zoom.us/)
- [Zoom App Marketplace](https://marketplace.zoom.us/)
- [Zoom Developer Forum](https://devforum.zoom.us/)
- [Zoom GitHub](https://github.com/zoom)
- [RUNBOOK.md](RUNBOOK.md) - repository maintenance and preflight checklist

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on improving this skill repository.

## License

MIT
