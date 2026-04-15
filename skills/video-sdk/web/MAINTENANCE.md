# Skill Maintenance Guide

## How This Skill Was Created

### 1. Information Sources

#### Type Definition Files (Primary Source)

```
./types/
├── index.d.ts          # Main exports
├── zoomvideo.d.ts      # ZoomVideo namespace
├── videoclient.d.ts    # VideoClient methods and events
├── media.d.ts          # MediaStream, audio/video options
├── common.d.ts         # Shared types (Participant, enums)
├── chat.d.ts           # Chat client
├── recording.d.ts      # Recording client
├── subsession.d.ts     # Breakout rooms
├── command.d.ts        # Command channel
├── preview.d.ts        # Preview/local tracks
├── live-transcription.d.ts
├── live-stream.d.ts
├── whiteboard.d.ts
├── broadcast-streaming.d.ts
├── real-time-media-streams.d.ts
└── event-callback.d.ts # All event callback types
```

#### Official Documentation URLs

```
# Main Documentation
https://developers.zoom.us/docs/video-sdk/web/get-started/
https://developers.zoom.us/docs/video-sdk/web/sessions/
https://developers.zoom.us/docs/video-sdk/web/event-handling/

# Audio & Video
https://developers.zoom.us/docs/video-sdk/web/video/
https://developers.zoom.us/docs/video-sdk/web/audio/
https://developers.zoom.us/docs/video-sdk/web/preview/

# Screen Sharing
https://developers.zoom.us/docs/video-sdk/web/share/
https://developers.zoom.us/docs/video-sdk/web/share-annotation/
https://developers.zoom.us/docs/video-sdk/web/share-browser-options/

# Advanced Features
https://developers.zoom.us/docs/video-sdk/web/recording/
https://developers.zoom.us/docs/video-sdk/web/subsessions/
https://developers.zoom.us/docs/video-sdk/web/live-stream/
https://developers.zoom.us/docs/video-sdk/web/command-channel/
https://developers.zoom.us/docs/video-sdk/web/transcription-translation/
https://developers.zoom.us/docs/video-sdk/web/chat/

# Phone Integration
https://developers.zoom.us/docs/video-sdk/web/pstn/
https://developers.zoom.us/docs/video-sdk/web/sip/

# Quality & Compatibility
https://developers.zoom.us/docs/video-sdk/web/quality/
https://developers.zoom.us/docs/video-sdk/web/browser-support/
https://developers.zoom.us/docs/video-sdk/web/sharedarraybuffer/
https://developers.zoom.us/docs/video-sdk/web/error-codes/

# Other
https://developers.zoom.us/docs/video-sdk/web/virtual-background/
https://developers.zoom.us/docs/video-sdk/web/frameworks/
```

### 2. File Structure

```
skills/video-sdk/web/
├── SKILL.md                   # Main file (required) - Quick Start, Core API
├── MAINTENANCE.md             # This file - maintenance guide
├── concepts/                  # Mental models that apply to every feature
│   ├── sdk-architecture-pattern.md   # Universal 5-step pattern
│   └── singleton-hierarchy.md        # ZoomVideo → sub-client navigation tree
└── references/                # Feature-specific how-to docs and framework guides
    ├── SESSIONS.md            # Session management
    ├── AUDIO.md               # Audio features
    ├── VIDEO.md               # Video features
    ├── SCREEN-SHARE.md        # Screen sharing
    ├── FEATURES.md            # Advanced features (recording, subsessions, streaming, etc.)
    ├── ADVANCED.md            # Chat, processors, whiteboard
    ├── BROWSER-SUPPORT.md     # Browser compatibility
    ├── ERROR-CODES.md         # Error code reference
    ├── COMMON-ISSUES.md       # Troubleshooting guide (symptom → cause → fix)
    ├── REACT.md               # React 19 framework guide
    ├── VUE.md                 # Vue 3 framework guide
    ├── ANGULAR.md             # Angular 21 framework guide
    └── SVELTE.md              # Svelte 5 framework guide
```

### 3. SKILL.md Format Requirements

```yaml
---
name: zoom-videosdk-web # Required: lowercase letters, numbers, hyphens only
description: Description text... # Required: max 1024 characters
---
# Title

Content...
```

## Update Process

### When SDK Version Updates

1. **Update Type Definitions**

   ```bash
   npm update @zoom/videosdk
   # or
   npm install @zoom/videosdk@latest
   ```

2. **Check Type Changes**

   ```bash
   # Compare old and new type definitions
   diff -r old-types/ types/
   ```

3. **Fetch Latest Documentation**

   - Visit the official documentation URLs listed above
   - Check changelog: https://developers.zoom.us/changelog/

4. **Update Skill Files**
   - Update version number in SKILL.md
   - Update relevant feature documentation
   - Add new APIs/features
   - Remove deprecated APIs

### Update Checklist

- [ ] Update `SKILL.md` version number (v2.4.0 → vX.X.X)
- [ ] Check `ZoomVideo` static method changes
- [ ] Check `VideoClient` method changes
- [ ] Check `MediaStream` method changes
- [ ] Check event name/payload changes
- [ ] Check new/removed Clients (Chat, Recording, etc.)
- [ ] Check new/removed enum types
- [ ] Check new/deprecated option parameters
- [ ] Update error codes (if any new ones)
- [ ] Update browser compatibility info

## Using Claude Code for Updates

### Auto-fetch Documentation Updates

```
Please help me update the zoom-videosdk-web skill:
1. Read all type definition files under ./types/
2. Fetch official docs: https://developers.zoom.us/docs/video-sdk/web/
3. Compare with existing skill files to find what needs updating
4. Update the relevant documentation
```

### Add New Feature Documentation

```
Please add [new feature name] documentation to the skill:
1. Fetch official docs: [URL]
2. Read relevant type definitions: ./types/[file].d.ts
3. Update the appropriate skill file
```

### Update Specific File

```
Please update skills/video-sdk/web/[FILE].md:
1. Read ./types/[relevant type file].d.ts
2. Fetch latest docs: [URL]
3. Update the content
```

## Version History

### v1.0.0 (2024-12-16)

- Initial version
- SDK Version: 2.4.0
- Features included:
  - Session management (create/join/leave)
  - Audio (start/mute/devices/noise suppression/PSTN)
  - Video (capture/render/virtual background/PTZ)
  - Screen sharing (share/annotation/tab audio)
  - Chat (messages/files)
  - Recording (cloud recording)
  - Subsessions (create/assign/broadcast)
  - Live streaming (RTMP)
  - Command channel (custom signaling)
  - Transcription (live transcription/translation)
  - Phone integration (PSTN/SIP/H.323)
  - Quality monitoring (stats/network quality)
  - Custom processors (audio/video/share)
  - Whiteboard

## Best Practices

1. **Keep it concise**: Skill files should provide quick reference, not complete documentation
2. **Code examples**: Every API should have runnable code examples
3. **Type accuracy**: Ensure type signatures match the `.d.ts` files
4. **Complete events**: Ensure all events are documented
5. **Error handling**: Include common errors and solutions
6. **Version annotations**: Mark new APIs with version (e.g., `@since 2.2.0`)
