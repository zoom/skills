# Screen Sharing Guide

## Overview

Screen sharing allows users to share their screen, window, or browser tab with other participants. Available on **desktop browsers only**.

## Starting Screen Share

```typescript
const stream = client.getMediaStream();

// Check which element type to use
if (stream.isStartShareScreenWithVideoElement()) {
  await stream.startShareScreen(videoElement);
} else {
  await stream.startShareScreen(canvasElement);
}

// With options
await stream.startShareScreen(element, {
  displaySurface: 'browser',          // 'browser' | 'window' | 'monitor'
  optimizedForSharedVideo: true,      // Better quality for video files
  broadcastToSubsession: true,        // Share to breakout rooms (host only)
  requestReadReceipt: true,           // Get 'share-can-see-screen' event
  simultaneousShareView: true,        // View others' shares while sharing
  hideShareAudioOption: false,        // Hide "Share Audio" checkbox
  controls: {
    monitorTypeSurfaces: 'include',   // 'include' | 'exclude'
    surfaceSwitching: 'include',      // Allow dynamic surface switching
    selfBrowserSurface: 'exclude',    // Exclude current tab
    systemAudio: 'include',           // Enable system audio sharing
    preferCurrentTab: false,          // Prefer current tab in picker
  }
});
```

### Display Surface Options

| Value | Description |
|-------|-------------|
| `'browser'` | Tabs only |
| `'window'` | Application windows |
| `'monitor'` | Entire screen |

### Share Options Reference

| Option | Type | Description |
|--------|------|-------------|
| `displaySurface` | string | Limit surface type in picker |
| `optimizedForSharedVideo` | boolean | Better quality for video content |
| `broadcastToSubsession` | boolean | Share to breakout rooms |
| `requestReadReceipt` | boolean | Receive view confirmation |
| `simultaneousShareView` | boolean | View others while sharing |
| `hideShareAudioOption` | boolean | Hide audio checkbox |
| `secondaryCameraId` | string | Share secondary camera |
| `captureWidth/Height` | number | Secondary camera resolution |
| `sourceId` | string | For Electron/NW.js apps |

### Privacy Controls (Chrome)

```typescript
controls: {
  monitorTypeSurfaces: 'include' | 'exclude',  // Full screen option
  surfaceSwitching: 'include' | 'exclude',     // Dynamic switching
  selfBrowserSurface: 'include' | 'exclude',   // Current tab
  systemAudio: 'include' | 'exclude',          // System audio
  preferCurrentTab: boolean,                    // Prefer current tab
}
```

## Stopping Screen Share

```typescript
// Stop programmatically
await stream.stopShareScreen();

// User stopped via browser controls
client.on('passively-stop-share', (payload) => {
  console.log('Share stopped by user or system');
});
```

## Viewing Shared Content

### Modern API (v2.2.10+) - Recommended

Supports up to **4 simultaneous shares**.

```typescript
// Attach share view
const element = await stream.attachShareView(userId);
document.getElementById('share-container').appendChild(element);

// Detach when done
await stream.detachShareView(userId);

// Listen for share changes
client.on('peer-share-state-change', async (payload) => {
  const { userId, action } = payload;

  if (action === 'Start') {
    const element = await stream.attachShareView(userId);
    document.getElementById('share-container').appendChild(element);
  } else if (action === 'Stop') {
    await stream.detachShareView(userId);
  }
});
```

### Legacy API

Single share view only.

```typescript
// Start viewing
await stream.startShareView(canvasElement, userId);

// Stop viewing
stream.stopShareView();

// Listen for active share
client.on('active-share-change', async (payload) => {
  const { userId, state } = payload;

  if (state === 'Active') {
    await stream.startShareView(canvas, userId);
  } else {
    stream.stopShareView();
  }
});
```

**Note**: Don't mix `attachVideo` and `attachShareView` in the same container.

## Share Privileges

Control who can share:

```typescript
// Get current privilege
const privilege = stream.getSharePrivilege();

// Set privilege (host/manager only)
await stream.setSharePrivilege(SharePrivilege.Unlocked);
await stream.setSharePrivilege(SharePrivilege.Locked);
await stream.setSharePrivilege(SharePrivilege.MultipleShare);
```

### SharePrivilege Enum

```typescript
enum SharePrivilege {
  Unlocked = 0,      // One at a time, host can override
  Locked = 1,        // Only host/manager can share
  MultipleShare = 3, // Multiple simultaneous shares
}
```

## Sharing Tab Audio

### Requirements

Tab audio sharing requires either:
1. **SharedArrayBuffer** enabled (WebAssembly mode)
2. **WebRTC mode** with `audio_webrtc_mode: 1` in JWT

### Enable Tab Audio

```typescript
await stream.startShareScreen(element, {
  controls: {
    systemAudio: 'include',  // Show audio option
  }
});

// User selects "Share tab audio" in browser picker
```

### Check Audio Support

```typescript
// Check if mic and share audio can work together
const canBoth = stream.isSupportMicrophoneAndShareAudioSimultaneously();
// Chrome 111+: true
// Chrome < 111: false (must toggle between sources)
```

### Control Share Audio

```typescript
// Get status
const status = stream.getShareAudioStatus();
// { isShareAudioEnabled, isShareAudioMuted, isSharingAudio }

// Mute/unmute share audio
await stream.muteShareAudio();
await stream.unmuteShareAudio();
```

### Share Audio Event

```typescript
client.on('share-audio-change', (payload) => {
  const { state } = payload;
  // Track audio source changes
});
```

## Secondary Camera Share

Share a document camera or secondary webcam:

```typescript
const cameras = stream.getCameraList();
const documentCamera = cameras.find(c => c.label.includes('Document'));

await stream.startShareScreen(element, {
  secondaryCameraId: documentCamera.deviceId,
  captureWidth: 1920,
  captureHeight: 1080,
});
```

## Share Events

### Peer Share State Change

```typescript
client.on('peer-share-state-change', (payload) => {
  const { userId, action } = payload;
  // action: 'Start' | 'Stop'
});
```

### Active Share Change

```typescript
client.on('active-share-change', (payload) => {
  const { userId, state } = payload;
  // state: 'Active' | 'Inactive'
});
```

### Share Content Change

```typescript
client.on('share-content-change', (payload) => {
  // Share content type changed
});
```

### Share Dimension Change

```typescript
client.on('share-content-dimension-change', (payload) => {
  const { userId, width, height } = payload;
  // Adjust container size
});
```

### Share Privilege Change

```typescript
client.on('share-privilege-change', (payload) => {
  const { privilege } = payload;
  // Update UI based on new privilege
});
```

### Passively Stop Share

```typescript
client.on('passively-stop-share', (payload) => {
  // Share stopped via browser controls or system
});
```

### Share Can See Screen

```typescript
// Requires requestReadReceipt: true
client.on('share-can-see-screen', (payload) => {
  const { oderId } = payload;
  console.log(`User ${oderId} can now see the share`);
});
```

## Annotation

Enable drawing/highlighting on shared content.

### Setup

```html
<!-- Container must be position: relative -->
<div id="share-container" style="position: relative;">
  <canvas id="share-canvas"></canvas>
</div>
```

### Start Annotation

```typescript
// After starting share or viewing share
await stream.startAnnotation();

// Get annotation controller
const controller = stream.getAnnotationController();
```

### Check Permission

```typescript
// Check if annotation is allowed
const canAnnotate = stream.canDoAnnotation();
```

### Annotation Controller

```typescript
const controller = stream.getAnnotationController();

// Set tool type
controller.setToolType('pen');      // 'pen' | 'eraser' | 'vanishingPen' | ...

// Set color (RGBA hex or CSS)
controller.setToolColor('#ff0000');
controller.setToolColor(0xff0000ff);

// Set line width (default: 8px)
controller.setToolWidth(4);

// Undo/Redo
controller.undo();
controller.redo();

// Clear annotations
controller.clear('own');     // Clear own annotations
controller.clear('viewer');  // Clear viewers' (presenter only)
controller.clear('all');     // Clear all (presenter/host only)
```

### Annotation Privileges

```typescript
// Presenter controls viewer annotation
await stream.changeAnnotationPrivilege(true);  // Allow viewers
await stream.changeAnnotationPrivilege(false); // Disallow viewers
```

### Annotation Events

```typescript
// Viewer requests to draw
client.on('annotation-viewer-draw-request', (payload) => {
  const { userId, displayName } = payload;
  // Approve or deny request
});

// Privilege changed
client.on('annotation-privilege-change', (payload) => {
  const { canAnnotate } = payload;
  // Update UI
});

// Undo/Redo availability
client.on('annotation-undo-status', (payload) => {
  const { canUndo } = payload;
});

client.on('annotation-redo-status', (payload) => {
  const { canRedo } = payload;
});
```

### Stop Annotation

```typescript
// When share pauses or ends
await stream.stopAnnotation();
```

## Share Statistics

```typescript
// Subscribe
stream.subscribeShareStatisticData();

client.on('share-statistic-data-change', (payload) => {
  const { encoding, decoding } = payload.data;
  console.log('Share FPS:', encoding.fps);
  console.log('Share Resolution:', `${encoding.width}x${encoding.height}`);
});

// Unsubscribe
stream.unsubscribeShareStatisticData();
```

## Error Codes

| Code | Name | Description |
|------|------|-------------|
| 6200 | SCREEN_SHARE_USER_DENIED | User cancelled share picker |
| 6201 | SCREEN_SHARE_SYSTEM_DENIED | System permission denied |
| 6204 | SCREEN_SHARE_HOST_PRIVILEGE_REQUIRED | Share locked to host |
| 6210 | SCREEN_SHARE_IN_PROGRESS | Already sharing |

## Best Practices

1. **Check element type** with `isStartShareScreenWithVideoElement()` before starting
2. **Use modern API** (`attachShareView`/`detachShareView`) for multiple share support
3. **Handle passive stop** to clean up when user stops via browser
4. **Set appropriate privilege** based on meeting needs
5. **Enable `simultaneousShareView`** to view others while sharing
6. **Restart annotation** when starting a new share session
7. **Use position: relative** container for annotation overlay
