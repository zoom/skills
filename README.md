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
   - [skills/plugin-sdk/SKILL.md](skills/plugin-sdk/SKILL.md)
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
| "Control Zoom Workplace from my desktop app" | zoom-plugin-sdk |
| "Handle webhook events" | webhooks |
| "Build a meeting bot" | zoom-meeting-sdk + rtms |
| "Set up OAuth authentication" | oauth |
| "Summarize this transcript" | summarizer |
| "Translate this text" | translator |
| "Build AI-agent meeting search tools" | zoom-mcp |
| "Manage Zoom Tasks from an AI agent" | zoom-mcp/tasks |
| "Analyze Revenue Accelerator conversations" | zoom-mcp/revenue-accelerator |

### 3. Skills chain automatically

When your task requires multiple skills, the agent loads them as needed. For example, "build a meeting bot" loads:
- **zoom-meeting-sdk** (for joining meetings)
- **rtms** (for real-time audio/video/transcript access)
- **zoom-rest-api** (for creating meetings)

## Test Marketplace Apps Locally

When using the Marketplace helper MCP server to create or update a development app, you can
test without deploying a public server first. Run your local app on HTTP, then expose it through
an HTTPS tunnel. Use the tunnel URL for the development OAuth redirect URL, Home URL, and webhook
or bot endpoint fields.

### ngrok (recommended)

```bash
# Your local app should listen on port 3000.
ngrok http 3000
```

Copy the HTTPS forwarding URL, for example `https://example.ngrok-free.app`, and configure routes
such as:

```text
https://example.ngrok-free.app/                 # development Home URL
https://example.ngrok-free.app/oauth/callback   # OAuth redirect URL
https://example.ngrok-free.app/webhooks/zoom    # webhook or bot endpoint
```

After the tunnel is live, use `app-builder-for-agents` MCP to create or update the development
app with the tunnel-derived URLs. Update the development OAuth redirect URL and allow-list entry,
Home URL, and webhook or bot endpoint as applicable. The redirect URL must match the OAuth request
exactly. Free tunnel URLs can change after restart, so run the helper update again before testing.
Do not put temporary tunnel URLs in production fields.

Configure the helper separately in each MCP client:

Codex:
```bash
codex mcp add app-builder-for-agents \
  --url https://d3k9b5xygup21i.cloudfront.net/mcp \
  --oauth-resource https://d3k9b5xygup21i.cloudfront.net/mcp
```

Claude Code:
```bash
claude mcp add --transport http \
  app-builder-for-agents \
  https://6a61-38-99-100-21.ngrok-free.app/mcp
```

The Claude Code URL is a tunnel endpoint and may change. Re-add or update the server when the
tunnel URL changes. Do not add this helper to the repository `.mcp.json`; that file contains the
bundled Zoom product MCP servers, while helper endpoints and client authentication are environment
specific.

### Cloudflare Tunnel (alternative)

```bash
cloudflared tunnel --url http://localhost:3000
```

Use the generated HTTPS URL the same way. A named Cloudflare Tunnel is preferable when you need a
stable development hostname. In both cases, keep the app in development mode, validate webhook
signatures, and do not expose client secrets or access tokens through the tunneled app. See the
[Marketplace local tunnel workflow](skills/rest-api/references/marketplace-apps.md#local-tunnel-workflow-with-app-builder-for-agents)
for the exact update sequence.
For external REST or MCP connectors and manifest-managed actions/triggers, use the
[Marketplace feature fragments](skills/rest-api/references/marketplace-connect-actions-triggers.md)
and merge them into a complete General App manifest rather than submitting them alone.

## Skills

| Skill | Description |
|-------|-------------|
| [zoom-general](skills/general/) | **Hub** - Core concepts, authentication, use cases, routing |
| [zoom-rest-api](skills/rest-api/) | 600+ REST API endpoints, rate limits, pagination |
| [zoom-webhooks](skills/webhooks/) | Real-time event notifications |
| [zoom-websockets](skills/websockets/) | Real-time WebSocket event connections |
| [zoom-meeting-sdk](skills/meeting-sdk/) | Embed Zoom meetings (Web, React Native, Electron, Linux headless bots) |
| [zoom-video-sdk](skills/video-sdk/) | Custom video experiences (Web, React Native, Flutter, Linux headless bots) |
| [zoom-plugin-sdk](skills/plugin-sdk/) | Native macOS/Windows companion apps that control installed Zoom Workplace over IPC |
| [zoom-apps-sdk](skills/zoom-apps-sdk/) | Apps that run inside Zoom client |
| [scribe](skills/scribe/) | AI Services Scribe for uploaded-file and batch archive transcription |
| [summarizer](skills/summarizer/) | AI Services Summarizer for transcript recaps, summaries, and action items |
| [translator](skills/translator/) | AI Services Translator for plain-text and batch file translation |
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
| [zoom-mcp](skills/zoom-mcp/) | Parent router for the unified Zoom MCP Server and dedicated product MCP servers |
| [zoom-mcp/meetings](skills/zoom-mcp/meetings/) | Meeting search, assets, recordings, and recording-resource retrieval |
| [zoom-mcp/canvas](skills/zoom-mcp/canvas/) | Canvas file, block, collaborator, and content tools |
| [zoom-mcp/tasks](skills/zoom-mcp/tasks/) | Task, comment, assignee, collaborator, and step workflows |
| [zoom-mcp/revenue-accelerator](skills/zoom-mcp/revenue-accelerator/) | ZRA conversations, transcripts, analyses, deals, customers, and coaching data |
| [zoom-mcp/team-chat](skills/zoom-mcp/team-chat/) | Team Chat message, file, contact, session, channel, and member tools |
| [zoom-mcp/whiteboard](skills/zoom-mcp/whiteboard/) | Whiteboard creation, retrieval, and collaborator tools |

### Current Zoom MCP Servers

The current hosted catalog uses Streamable HTTP. Tool counts are observations, not API
contracts; run `tools/list` before relying on a cached inventory.

| Server | Endpoint |
|--------|----------|
| Zoom MCP Server | `https://mcp.zoom.us/mcp/zoom/streamable` |
| Meetings MCP | `https://mcp.zoom.us/mcp/meeting/streamable` |
| Chat MCP | `https://mcp.zoom.us/mcp/chat/streamable` |
| Canvas MCP | `https://mcp.zoom.us/mcp/canvas/streamable` |
| Tasks MCP | `https://mcp.zoom.us/mcp/tasks/streamable` |
| Whiteboard MCP | `https://mcp.zoom.us/mcp/whiteboard/streamable` |
| Revenue Accelerator MCP | `https://mcp.zoom.us/mcp/revenue_accelerator/streamable` |

## Common Use Cases

| Use Case | Skills Needed |
|----------|---------------|
| Schedule meetings programmatically | zoom-rest-api |
| Build meeting bots (AI/transcription) | zoom-meeting-sdk + rtms |
| Embed meetings in your app | zoom-meeting-sdk |
| Custom video experiences | zoom-video-sdk |
| Native desktop companion controls for Zoom Workplace | zoom-plugin-sdk + oauth |
| Auto-download recordings to S3/GCS | webhooks + zoom-rest-api |
| Real-time AI processing | rtms |
| Batch or on-demand media transcription | scribe |
| Transcript summarization and action items | summarizer |
| Text translation and localization | translator |
| In-meeting collaborative apps | zoom-apps-sdk |
| Team Chat integrations | zoom-team-chat |
| Virtual Agent campaign/chat flows (web + mobile wrappers) | virtual-agent + contact-center |
| Contact Center app/web/mobile integrations | contact-center |
| Rivet-based event-driven API backend | rivet-sdk + oauth + zoom-rest-api |
| Pre-join/browser readiness diagnostics | probe-sdk + meeting-sdk or video-sdk |
| Low-latency event notifications | zoom-websockets |
| OAuth authentication setup | oauth |
| AI-driven tool workflows over Zoom meetings, Team Chat, Canvas, and recordings | zoom-mcp |
| Team Chat MCP search, read, and write workflows | zoom-mcp/team-chat |
| Meeting asset and recording retrieval through dedicated MCP | zoom-mcp/meetings |
| Zoom Canvas file and block workflows through dedicated MCP | zoom-mcp/canvas |
| Task management through MCP | zoom-mcp/tasks |
| Revenue Accelerator conversation and deal intelligence | zoom-mcp/revenue-accelerator |
| Enterprise AI architecture (API core + AI tool layer) | zoom-rest-api + zoom-mcp |

## Current Package Snapshot

Public package versions verified on 2026-07-10:

| Package | Version |
|---------|---------|
| `@zoom/meetingsdk` | `6.2.0` |
| `@zoom/meetingsdk-react-native` | `7.0.5` |
| `@zoom/videosdk` | `2.4.5` |
| `@zoom/react-native-videosdk` | `2.5.10` |
| `flutter_zoom_videosdk` | `2.5.10` |
| `@zoom/videosdk-ui-toolkit` | `2.4.5-1` |
| `@zoom/appssdk` | `0.16.39` |
| `@zoom/probesdk` | `1.0.4` |
| `@zoom/rivet` | `0.4.0` |

Use [the SDK upgrade guide](skills/general/references/sdk-upgrade-guide.md) for source hierarchy,
version verification, and migration guardrails. Native SDK versions must be checked against the
download package or release notes for the target platform rather than inferred from web packages.

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
       ├── zoom-plugin-sdk
       ├── zoom-apps-sdk
       ├── scribe
       ├── summarizer
       ├── translator
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
           ├── whiteboard
           ├── team-chat
           ├── meetings
           ├── canvas
           ├── tasks
           └── revenue-accelerator
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
│   ├── plugin-sdk/
│   │   ├── macos/
│   │   └── windows/
│   ├── zoom-apps-sdk/
│   ├── scribe/
│   ├── summarizer/
│   ├── translator/
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
│       ├── whiteboard/
│       ├── team-chat/
│       ├── meetings/
│       ├── docs/
│       ├── tasks/
│       └── revenue-accelerator/
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
