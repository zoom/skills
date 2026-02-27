# Zoom Developer Platform Skills

Skills for building with Zoom SDKs, APIs, and integrations. Focused on web development.

Primary skill entrypoint: [SKILL.md](SKILL.md)

## Installation

### Codex

Codex loads skills from `~/.codex/skills/`. This repo is a single skill folder (it contains the root `SKILL.md`), so you can install it as one directory.

Option A: clone directly into Codex skills:
```bash
git clone https://github.com/zoom/skills.git ~/.codex/skills/zoom-skills
```

Option B: keep a working copy and symlink it (recommended for easy updates):
```bash
git clone https://github.com/zoom/skills.git ~/zoom-skills
ln -s ~/zoom-skills ~/.codex/skills/zoom-skills
```

If you previously installed it under a different folder name (for example `agent-skills`), remove/replace the old folder so Codex doesn’t load duplicates:
```bash
rm -rf ~/.codex/skills/agent-skills
```

### Claude Code

```bash
# Clone the repository
git clone https://github.com/zoom/skills.git

# Copy to Claude Code skills directory
cp -r skills ~/.claude/skills/zoom-skills
```

Or install individual skills:
```bash
cp -r skills/zoom-general ~/.claude/skills/
cp -r skills/zoom-meeting-sdk ~/.claude/skills/
```

### OpenCode

```bash
# Copy to OpenCode skills directory
cp -r skills ~/.config/opencode/skills/zoom-skills
```

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

### 3. Skills chain automatically

When your task requires multiple skills, the agent loads them as needed. For example, "build a meeting bot" loads:
- **zoom-meeting-sdk** (for joining meetings)
- **rtms** (for real-time audio/video/transcript access)
- **zoom-rest-api** (for creating meetings)

## Skills

| Skill | Description |
|-------|-------------|
| [zoom-general](general/) | **Hub** - Core concepts, authentication, use cases, routing |
| [zoom-rest-api](rest-api/) | 600+ REST API endpoints, rate limits, pagination |
| [zoom-webhooks](webhooks/) | Real-time event notifications |
| [zoom-websockets](websockets/) | Real-time WebSocket event connections |
| [zoom-meeting-sdk](meeting-sdk/) | Embed Zoom meetings (Web, React Native, Electron, Linux headless bots) |
| [zoom-video-sdk](video-sdk/) | Custom video experiences (Web, React Native, Flutter, Linux headless bots) |
| [zoom-apps-sdk](zoom-apps-sdk/) | Apps that run inside Zoom client |
| [zoom-rtms](rtms/) | Real-time Media Streams (live audio/video/transcripts) |
| [zoom-team-chat](team-chat/) | Team Chat APIs and integrations |
| [virtual-agent](virtual-agent/) | Virtual Agent web embeds, Android/iOS wrappers, and KB sync workflows |
| [contact-center](contact-center/) | Contact Center apps, web embeds, and Android/iOS native SDKs |
| [phone](phone/) | Zoom Phone APIs, Smart Embed, URI schemes, and webhook patterns |
| [rivet-sdk](rivet-sdk/) | Rivet JavaScript server framework for auth, webhooks, and typed endpoint wrappers |
| [probe-sdk](probe-sdk/) | Browser/device/network readiness diagnostics before Meeting SDK or Video SDK joins |
| [zoom-ui-toolkit](ui-toolkit/) | Pre-built UI components for Video SDK |
| [zoom-cobrowse-sdk](cobrowse-sdk/) | Collaborative browsing for support |
| [zoom-oauth](oauth/) | OAuth authentication (all 4 grant types) |

## Common Use Cases

| Use Case | Skills Needed |
|----------|---------------|
| Schedule meetings programmatically | zoom-rest-api |
| Build meeting bots (AI/transcription) | zoom-meeting-sdk + rtms |
| Embed meetings in your app | zoom-meeting-sdk |
| Custom video experiences | zoom-video-sdk |
| Auto-download recordings to S3/GCS | webhooks + zoom-rest-api |
| Real-time AI processing | rtms |
| In-meeting collaborative apps | zoom-apps-sdk |
| Team Chat integrations | zoom-team-chat |
| Virtual Agent campaign/chat flows (web + mobile wrappers) | virtual-agent + contact-center |
| Contact Center app/web/mobile integrations | contact-center |
| Rivet-based event-driven API backend | rivet-sdk + oauth + zoom-rest-api |
| Pre-join/browser readiness diagnostics | probe-sdk + meeting-sdk or video-sdk |
| Low-latency event notifications | zoom-websockets |
| OAuth authentication setup | oauth |

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
       ├── zoom-rtms
       ├── zoom-team-chat
       ├── virtual-agent
       ├── contact-center
       ├── phone
       ├── rivet-sdk
       ├── probe-sdk
       ├── zoom-ui-toolkit
       ├── zoom-cobrowse-sdk
       └── zoom-oauth
```

## Directory Structure

```
agent-skills/
├── README.md                 # This file
├── ARCHITECTURE.md           # Full architecture diagram
│
├── general/            # HUB (entry point)
│   ├── SKILL.md
│   ├── references/           # Cross-cutting docs
│   └── use-cases/            # Multi-skill scenarios
│
└── zoom-*/                   # SPOKES (specialized skills)
    ├── SKILL.md
    └── references/
```

## Resources

- [Zoom Developer Platform](https://developers.zoom.us/)
- [Zoom App Marketplace](https://marketplace.zoom.us/)
- [Zoom Developer Forum](https://devforum.zoom.us/)
- [Zoom GitHub](https://github.com/zoom)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on improving this skill repository.

## License

MIT
