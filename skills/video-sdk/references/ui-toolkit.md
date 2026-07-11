# Video SDK UI Toolkit

Pre-built, low-code video chat UI powered by Zoom Video SDK.

## Overview

The Video SDK UI Toolkit provides ready-to-use video components so you don't have to build the UI from scratch. Available for Web, iOS, and Android.

## Platform Availability

| Platform | UI Toolkit | Package/Repo |
|----------|-----------|--------------|
| **Web** | Yes | `@zoom/videosdk-ui-toolkit` |
| **iOS** | Yes | [zoom/videosdk-ui-toolkit-ios](https://github.com/zoom/videosdk-ui-toolkit-ios) |
| **Android** | Yes | [zoom/videosdk-uitoolkit-android-sample](https://github.com/zoom/videosdk-uitoolkit-android-sample) |
| React Native | No | Use SDK + custom UI |
| Flutter | No | Use SDK + custom UI |

## Web Installation

### npm

```bash
npm install @zoom/videosdk-ui-toolkit --save
```

Current verified latest is `@zoom/videosdk-ui-toolkit@2.4.5-1` as of 2026-07-10. Its peer dependencies include `@zoom/videosdk@^2.4.5`, React 18, React DOM 18, `@reduxjs/toolkit@^2`, `react-redux@^9`, and `react-draggable@^4`.

Use `2.3.15-1` or newer for WebRTC video because Zoom flags that version floor for future Chrome compatibility.

```javascript
import uitoolkit from "@zoom/videosdk-ui-toolkit";
import "@zoom/videosdk-ui-toolkit/dist/videosdk-ui-toolkit.css";
```

### CDN

```html
<link rel="stylesheet" href="https://source.zoom.us/uitoolkit/2.4.5-1/videosdk-ui-toolkit.css" />
<script src="https://source.zoom.us/uitoolkit/2.4.5-1/videosdk-ui-toolkit.min.umd.js"></script>
<script>
  const uitoolkit = window.UIToolkit;
</script>
```

### Angular

Add CSS to `angular.json`:
```json
"styles": [
  "node_modules/@zoom/videosdk-ui-toolkit/dist/videosdk-ui-toolkit.css"
]
```

## Quick Start

```html
<div id="sessionContainer"></div>
```

```javascript
import uitoolkit from "@zoom/videosdk-ui-toolkit";
import "@zoom/videosdk-ui-toolkit/dist/videosdk-ui-toolkit.css";

const config = {
  videoSDKJWT: "your-jwt-token",
  sessionName: "SessionA",
  userName: "UserA",
  sessionPasscode: "abc123",
  featuresOptions: {
    preview: { enable: true },
    video: { enable: true },
    audio: { enable: true },
    share: { enable: true },
    chat: { enable: true, enableEmoji: true },
    users: { enable: true },
    settings: { enable: true },
    leave: { enable: true },
  },
};

const sessionContainer = document.getElementById("sessionContainer");
void uitoolkit.joinSession(sessionContainer, config);
```

## Feature Components

Toggle components on/off via `featuresOptions`:

| Component | Description | Paid Plan? |
|-----------|-------------|------------|
| `preview` | Pre-session device selection, virtual background | No |
| `video` | Video layout for sending/receiving | No |
| `audio` | Audio button and controls | No |
| `secondaryAudio` | Secondary audio support | No |
| `share` | Screen sharing | No |
| `chat` | In-session chat | No |
| `users` | Participant list | No |
| `settings` | Device config, virtual background, stats | No |
| `viewMode` | Grid/speaker view options | No |
| `leave` | Leave/end session button | No |
| `invite` | Invite via link | No |
| `theme` | Theme color selection | No |
| `feedback` | Session feedback form | No |
| `troubleshoot` | Zoom Probe SDK troubleshooting | No |
| `subsession` | Breakout rooms button | No |
| `playback` | Media file playback | No |
| `virtualBackground` | Custom and built-in virtual background choices | No |
| `footer` | Footer controls area | No |
| `header` | Session header area | No |
| `screenshot` | Video/share screenshot toggles | No |
| `recording` | Cloud recording | **Yes** |
| `phone` | Join by phone audio | **Yes** |
| `caption` | Live translations | **Yes** |

Use `featuresOptions` for new work. Current typings mark the older `features`, `options`, and top-level `virtualBackground` fields as deprecated.

Repo source also exposes advanced `featuresOptions` that are not all listed in the README table. Verify against the installed package types before shipping:

| Advanced option | Use |
|-----------------|-----|
| `video.ptz` | PTZ camera support for capable cameras |
| `audio.muteEntry` | Join session audio muted |
| `share.controls` | Supported browser display-media controls |
| `share.displaySurface` | Supported browser surface-selection hints |
| `share.hideShareAudioOption` | Hide share-computer-audio option where supported |
| `share.optimizedForSharedVideo` | Prefer frame rate for shared video content |
| `livestream` | Live stream UI support |
| `crc` | Room system / CRC invite support |
| `whiteboard` | Whiteboard UI and export flags |
| `rawData` | Source-level raw data configuration surface |
| `realTimeMediaStreams` | Source-level RTMS configuration surface |
| `cameraShare` | Source-level camera share entry point |

If TypeScript does not expose one of these options in the installed package, prefer the typed README/d.ts surface for production code or gate the source-only field behind a version check and local type augmentation.

## React Example

```jsx
import uitoolkit from "@zoom/videosdk-ui-toolkit";
import "@zoom/videosdk-ui-toolkit/dist/videosdk-ui-toolkit.css";
import { useEffect, useRef } from "react";

function VideoSession({ jwt, sessionName, userName }) {
  const containerRef = useRef(null);
  
  useEffect(() => {
    const config = {
      videoSDKJWT: jwt,
      sessionName,
      userName,
      sessionPasscode: "abc123",
      featuresOptions: {
        video: { enable: true },
        audio: { enable: true },
        chat: { enable: true, enableEmoji: true },
        users: { enable: true },
        leave: { enable: true },
      },
    };
    
    if (containerRef.current) {
      void uitoolkit.joinSession(containerRef.current, config);
    }
    
    return () => {
      if (containerRef.current) {
        void uitoolkit.closeSession(containerRef.current).then(() => uitoolkit.destroy());
      }
    };
  }, [jwt, sessionName, userName]);
  
  return <div ref={containerRef} style={{ height: '600px' }} />;
}
```

## Event Listeners

```javascript
// Session events
uitoolkit.onSessionJoined(() => {
  console.log("Session joined");
});

uitoolkit.onSessionClosed(() => {
  console.log("Session closed");
});

uitoolkit.onSessionDestroyed(() => {
  console.log("Toolkit DOM/resources destroyed");
});

uitoolkit.onViewTypeChange((viewType) => {
  console.log("View mode changed", viewType);
});

uitoolkit.onLanguageChange((language) => {
  console.log("Language changed", language);
});

// Unsubscribe
uitoolkit.offSessionJoined(callback);
uitoolkit.offSessionClosed(callback);
uitoolkit.offSessionDestroyed(callback);
```

## Component Visibility Control

```javascript
// Hide all components
uitoolkit.hideAllComponents();

// Show/hide individual components
uitoolkit.showControlsComponent(container);
uitoolkit.hideControlsComponent(container);

uitoolkit.showChatComponent(container);
uitoolkit.hideChatComponent(container);

uitoolkit.showUsersComponent(container);
uitoolkit.hideUsersComponent(container);

uitoolkit.showSettingsComponent(container);
uitoolkit.hideSettingsComponent(container);
```

The repo source/runtime bundle also includes broadcast viewer helpers:

```javascript
uitoolkit.showBroadcastViewerComponent(container, options);
uitoolkit.hideBroadcastViewerComponent(container);
```

These may not be present in every published `.d.ts`; verify the installed package before relying on TypeScript autocomplete.

## Close Session

```javascript
await uitoolkit.closeSession(sessionContainer);
await uitoolkit.destroy();
```

## Live Demo

- **Official live-hosted sample**: https://sdk.zoom.com/videosdk-uitoolkit

## Sample Repositories

| Framework | Repository |
|-----------|------------|
| React | [zoom/videosdk-ui-toolkit-react-sample](https://github.com/zoom/videosdk-ui-toolkit-react-sample) |
| Angular | [zoom/videosdk-ui-toolkit-angular-sample](https://github.com/zoom/videosdk-ui-toolkit-angular-sample) |
| Vue.js | [zoom/videosdk-ui-toolkit-vuejs-sample](https://github.com/zoom/videosdk-ui-toolkit-vuejs-sample) |
| Vanilla JS | [zoom/videosdk-ui-toolkit-javascript-sample](https://github.com/zoom/videosdk-ui-toolkit-javascript-sample) |

## Resources

- **Official docs**: https://developers.zoom.us/docs/video-sdk/web/ui-toolkit/
- **npm package**: https://www.npmjs.com/package/@zoom/videosdk-ui-toolkit
- **GitHub**: https://github.com/zoom/videosdk-ui-toolkit-web
