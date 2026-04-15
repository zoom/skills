# Video Guide

## Starting Video

Video requires a **user gesture** to start due to browser camera permission requirements.

```typescript
const stream = client.getMediaStream();

// Basic start
await stream.startVideo();

// With options
await stream.startVideo({
  cameraId: 'device-id',           // Specific camera
  hd: true,                        // 720p resolution
  fullHd: true,                    // 1080p resolution (Chrome/Edge only)
  mirrored: true,                  // Mirror self-view
  originalRatio: false,            // Keep original ratio (default: crop to 16:9)
  fps: 24,                         // Max FPS (10-30, default: 24)
  ptz: true,                       // Enable PTZ camera control
  virtualBackground: {
    imageUrl: 'https://example.com/bg.jpg',
    cropped: true,
  },
});
```

### Capture Video Options

| Option | Type | Description |
|--------|------|-------------|
| `cameraId` | string | Camera device ID or `MobileVideoFacingMode` |
| `hd` | boolean | Enable 720p capture |
| `fullHd` | boolean | Enable 1080p capture (Chrome/Edge only) |
| `mirrored` | boolean | Mirror self video |
| `captureWidth` | number | Custom capture width |
| `captureHeight` | number | Custom capture height |
| `fps` | number | Max frames per second (10-30) |
| `originalRatio` | boolean | Keep original aspect ratio |
| `ptz` | boolean | Enable pan-tilt-zoom |
| `virtualBackground` | object | Virtual background settings |
| `mask` | MaskOption | Video mask/frame settings |
| `mediaFile` | MediaPlaybackFile | Use media file as video input |

## Rendering Video

**CRITICAL**: You must use the `video-player-container` custom element for rendering video. The `attachVideo()` method returns an element that must be appended to this container.

**CRITICAL — Custom element default display:** Both `<video-player-container>` and `<video-player>` are custom elements that default to `display: inline` with **zero dimensions**. Without explicit CSS, `startVideo()` + `attachVideo()` both succeed silently but render nothing — you'll see a black area, no console errors, camera permission granted, and the participant's `bVideoOn` flag is `true`. Add these global CSS rules once in your app stylesheet:

```css
video-player-container {
  display: grid;
  width: 100%;
  height: 100%;
}
video-player {
  display: block;
  width: 100%;
  height: 100%;
  min-height: 200px;
}
```

Inline `style="width: 100%; height: 100%"` on a single container works too, but global rules are more reliable because the SDK may insert additional `<video-player>` children that also need sizing. Applying only Tailwind/utility classes on the container element is **not** sufficient — the base `display` needs to be overridden at the element-type level.


**CRITICAL — Non-SAB / Non-WebRTC fallback path:** When the browser has neither `SharedArrayBuffer` (COOP/COEP not set, or cross-origin isolation disabled) **nor** the WebRTC render path available, the SDK falls back to a legacy rendering mode that **cannot share a single `<video-player-container>` for both the self (send) stream and remote (receive) streams** — the canvas/element used for outgoing capture and the one used for incoming decode live on different pipelines and will clobber each other if combined.

In this mode you **must** maintain **two separate `<video-player-container>` elements**:

1. **Send container** — dedicated to the local user (`client.getCurrentUserInfo().userId`). Attach only the self video here via `stream.attachVideo(selfUserId, ...)`.
2. **Receive container** — used for all remote participants. Create one per remote user (or manage a pool) and attach each remote stream here.

Do **not** try to render the self video inside the same container as a remote tile, and do not swap the self tile into the gallery grid when in fallback mode — you'll see one of the streams go black, freeze, or throw a video-attach error.

**Two orthogonal toggles decide which render path the SDK uses:**

1. **SAB (SharedArrayBuffer) Gallery View** — client-side. Available when the page is cross-origin isolated (`COOP: same-origin` + `COEP: require-corp`) and the browser exposes `SharedArrayBuffer`. This is the default/preferred path.
2. **WebRTC(Gallery View)** — **hard-coded into the JWT signature payload** by your token-signing server. You opt in by setting this claim on the JWT you pass to `client.join()`:

   ```jsonc
   {
     // ...standard Video SDK claims (app_key, tpc, role_type, etc.)...
     "video_webrtc_mode": 1  // 1 = use WebRTC for video, 0/absent = use SAB/legacy
   }
   ```

   This is a **server-side flag** — the client cannot detect or change it at runtime; whatever the JWT says is what you get for the whole session.

**When fallback (two-container mode) is required:**

| SAB available? | `video_webrtc_mode` in JWT | Render mode | Can self + remotes share one container? |
|---|---|---|---|
| Yes | — | SAB(gallery view) | ✅ Yes |
| No | `1` | WebRTC(gallery View) | ✅ Yes |
| **No** | **`0` / absent** | **Legacy fallback(one send+ one receive)** | **❌ No — need two containers** |

So the fallback path is triggered **only** when the page is **not** cross-origin isolated **and** the JWT does not set `video_webrtc_mode: 1`. In that case you must split self (send) and remote (receive) into two separate `<video-player-container>` elements as described above.

**Practical rule:** if you can't guarantee cross-origin isolation headers (e.g. embedding in a third-party site), have your token server stamp `video_webrtc_mode: 1` on every JWT. That forces the WebRTC video path and you never hit the two-container fallback.

> This only affects older browser configurations or deployments that can't set the `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: require-corp` headers (see [references/browser-support.md](browser-support.md)). Production deployments should ship those headers so this path is never hit.

### Render Self Video

```typescript
// Start video capture
await stream.startVideo();

// Get container element (must be <video-player-container>)
const userId = client.getCurrentUserInfo().userId;
const container = document.getElementById('self-video') as HTMLElement;

// attachVideo returns an element to append to the container
const videoElement = await stream.attachVideo(userId, VideoQuality.Video_720P);

if (videoElement) {
  container.innerHTML = '';
  (videoElement as HTMLElement).style.width = '100%';
  (videoElement as HTMLElement).style.height = '100%';
  container.appendChild(videoElement as HTMLElement);
}
```

**HTML structure:**
```html
<video-player-container id="self-video" style="width: 100%; height: 100%;"></video-player-container>
```

### Render Other Participants

```typescript
// Listen for video state changes
client.on('peer-video-state-change', async (payload) => {
  const { userId, action } = payload;

  if (action === 'Start') {
    // Create video-player-container for user
    const container = document.createElement('video-player-container');
    container.id = `video-${userId}`;
    container.style.width = '100%';
    container.style.height = '100%';
    document.getElementById('video-grid')?.appendChild(container);

    // attachVideo returns element to append
    const videoElement = await stream.attachVideo(userId, VideoQuality.Video_360P);
    if (videoElement) {
      (videoElement as HTMLElement).style.width = '100%';
      (videoElement as HTMLElement).style.height = '100%';
      container.appendChild(videoElement as HTMLElement);
    }
  } else if (action === 'Stop') {
    // Detach video and remove container
    await stream.detachVideo(userId);
    document.getElementById(`video-${userId}`)?.remove();
  }
});
```

### TypeScript Custom Element Declaration

For TypeScript projects, declare the custom elements:

```typescript
// src/types/zoom-elements.d.ts
declare namespace JSX {
  interface IntrinsicElements {
    'video-player-container': React.DetailedHTMLProps<
      React.HTMLAttributes<HTMLElement>,
      HTMLElement
    >;
    'video-player': React.DetailedHTMLProps<
      React.HTMLAttributes<HTMLElement>,
      HTMLElement
    >;
  }
}
```

## Mobile Video

On mobile browsers, video rendering works differently. Self-view can use a standard `<video>` tag, while other participants still require `video-player-container`.

### Detect Mobile

```typescript
const isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
  navigator.userAgent
);

// Or use SDK method
const isSupportWebCodecs = typeof (window as any).VideoEncoder === 'function';
```

### Mobile Self-View (Video Tag)

On mobile, self-view can render directly to a `<video>` element:

```typescript
// Start video with mobile facing mode
await stream.startVideo({
  cameraId: MobileVideoFacingMode.User,  // Front camera
  mirrored: true,
});

// Render self-view to video element
const selfVideo = document.getElementById('self-video') as HTMLVideoElement;
const userId = client.getCurrentUserInfo().userId;

// For mobile self-view, pass the video element directly
await stream.attachVideo(userId, VideoQuality.Video_360P, selfVideo);
```

**HTML structure for mobile self-view:**
```html
<video id="self-video" autoplay playsinline muted style="width: 100%; height: 100%;"></video>
```

**Important attributes:**
- `autoplay` - Start playing automatically
- `playsinline` - Prevent fullscreen on iOS
- `muted` - Required for autoplay on mobile

### Mobile Other Participants

Other participants still require `video-player-container`:

```typescript
client.on('peer-video-state-change', async (payload) => {
  const { userId, action } = payload;

  if (action === 'Start') {
    const container = document.createElement('video-player-container');
    container.id = `video-${userId}`;
    container.style.width = '100%';
    container.style.height = '100%';
    document.getElementById('video-grid')?.appendChild(container);

    const videoElement = await stream.attachVideo(userId, VideoQuality.Video_360P);
    if (videoElement) {
      (videoElement as HTMLElement).style.width = '100%';
      (videoElement as HTMLElement).style.height = '100%';
      container.appendChild(videoElement as HTMLElement);
    }
  } else if (action === 'Stop') {
    await stream.detachVideo(userId);
    document.getElementById(`video-${userId}`)?.remove();
  }
});
```

### Mobile Camera Switching

```typescript
// Switch to front camera
await stream.switchCamera(MobileVideoFacingMode.User);

// Switch to back camera
await stream.switchCamera(MobileVideoFacingMode.Environment);
```

### Mobile Limitations

| Feature | Support |
|---------|---------|
| Max concurrent videos | 9 |
| Max quality | 720p |
| Virtual background | Not supported |
| Screen sharing | Limited (iOS Safari, Android Chrome) |
| PTZ camera control | Not supported |

### Mobile Best Practices

1. **Use lower quality** - `Video_360P` recommended for gallery view
2. **Limit video tiles** - Show max 4-6 videos for performance
3. **Use facing modes** - `MobileVideoFacingMode` instead of device IDs
4. **Add playsinline** - Required for iOS to prevent fullscreen
5. **Handle orientation** - Listen for orientation changes and adjust layout
6. **Test on real devices** - Emulators don't reflect real performance

### Active Speaker Detection

```typescript
client.on('video-active-change', (payload) => {
  // Fires when someone speaks for > 1 second
  const { userId, state } = payload;

  if (state === 'Active') {
    // Highlight this user's video
    highlightSpeaker(userId);
  } else {
    // Remove highlight
    unhighlightSpeaker(userId);
  }
});
```

## Video Quality

```typescript
enum VideoQuality {
  Video_90P = 0,   // Thumbnail
  Video_180P = 1,  // Low
  Video_360P = 2,  // Medium
  Video_720P = 3,  // HD
  Video_1080P = 4, // Full HD (Chrome/Edge only)
}
```

### Rendering Limits

| Platform | Max Concurrent Videos | Max Quality |
|----------|----------------------|-------------|
| Desktop | 25 | 720p |
| Mobile | 9 | 720p |

**Note**: Without SharedArrayBuffer, limit is 4 videos unless `enforceMultipleVideos: { disableRenderLimits: true }` is set.

## Stop Video

```typescript
// Stop video capture
await stream.stopVideo();

// Detach video element
await stream.detachVideo(userId);
```

## Camera Management

### List Cameras

```typescript
const cameras = stream.getCameraList();
// [{ label: 'FaceTime HD Camera', deviceId: '...' }, ...]
```

### Switch Camera

```typescript
// Desktop: by device ID
await stream.switchCamera('new-device-id');

// Mobile: by facing mode
await stream.switchCamera(MobileVideoFacingMode.User);        // Front camera
await stream.switchCamera(MobileVideoFacingMode.Environment); // Back camera
```

### Mobile Facing Modes

```typescript
enum MobileVideoFacingMode {
  User = 'user',             // Front camera
  Environment = 'environment', // Back camera
  Left = 'left',             // Left camera
  Right = 'right',           // Right camera
}
```

## Virtual Background

### Enable at Start

```typescript
await stream.startVideo({
  virtualBackground: {
    imageUrl: 'https://example.com/background.jpg',
    cropped: true,  // Crop to 16:9
  }
});
```

### Blur Background

```typescript
await stream.startVideo({
  virtualBackground: {
    imageUrl: 'blur',
  }
});
```

### Update During Session

```typescript
// Change background
await stream.updateVirtualBackgroundImage('https://example.com/new-bg.jpg');

// Switch to blur
await stream.updateVirtualBackgroundImage('blur');

// Remove virtual background
await stream.updateVirtualBackgroundImage(undefined);
```

### Requirements

- **Desktop only** - not available on mobile
- **SharedArrayBuffer required** for Firefox/Safari
- Chrome/Edge work without SAB using `enforceVirtualBackground: true`

```typescript
// Init option for browsers without SAB
await client.init('en-US', 'Global', {
  enforceVirtualBackground: true,  // Use canvas rendering
});
```

### Preload Virtual Background

```typescript
// Preload VB assets for faster initial load
client.on('video-virtual-background-preload-change', (payload) => {
  if (payload.status === 'success') {
    console.log('VB assets preloaded');
  }
});
```

## Video Mask

Apply a mask/frame around the video:

```typescript
await stream.startVideo({
  mask: {
    imageUrl: 'https://example.com/frame.png',
    cropped: true,
    rootWidth: 1280,
    rootHeight: 720,
    clip: {
      type: 'circle',
      radius: 200,
      x: 640,    // Center X
      y: 360,    // Center Y
    }
  }
});
```

### Clip Shapes

```typescript
// Circle
{ type: 'circle', radius: 200, x: 640, y: 360 }

// Square
{ type: 'square', length: 400, x: 640, y: 360 }

// Rectangle
{ type: 'rectangle', width: 400, height: 300, x: 640, y: 360 }

// SVG
{ type: 'svg', svg: 'https://example.com/mask.svg', width: 400, height: 300, x: 640, y: 360 }
```

## Video Events

### Peer Video State Change

```typescript
client.on('peer-video-state-change', (payload) => {
  const { userId, action } = payload;
  // action: 'Start' | 'Stop'
});
```

### Video Active Change (Speaker)

```typescript
client.on('video-active-change', (payload) => {
  const { userId, state } = payload;
  // state: 'Active' | 'Inactive'
});
```

### Video Dimension Change

```typescript
client.on('video-dimension-change', (payload) => {
  const { userId, width, height } = payload;
  // Adjust UI for new dimensions
});
```

### Video Capturing Change

```typescript
client.on('video-capturing-change', (payload) => {
  const { state } = payload;
  // state: 'Started' | 'Stopped' | 'Failed'
});
```

## Video Statistics

```typescript
// Subscribe
stream.subscribeVideoStatisticData();

client.on('video-statistic-data-change', (payload) => {
  const { encoding, decoding } = payload.data;

  console.log('Send FPS:', encoding.fps);
  console.log('Send Resolution:', `${encoding.width}x${encoding.height}`);
  console.log('Send Bitrate:', encoding.bitrate);

  console.log('Receive FPS:', decoding.fps);
  console.log('Receive Bitrate:', decoding.bitrate);
});

// Unsubscribe
stream.unsubscribeVideoStatisticData();
```

## PTZ Camera Control

Control pan-tilt-zoom cameras:

```typescript
// Start video with PTZ enabled
await stream.startVideo({ ptz: true });

// Control local camera
await stream.controlCamera(CameraControlCmd.ZoomIn);
await stream.controlCamera(CameraControlCmd.ZoomOut);
await stream.controlCamera(CameraControlCmd.Left);
await stream.controlCamera(CameraControlCmd.Right);
await stream.controlCamera(CameraControlCmd.Up);
await stream.controlCamera(CameraControlCmd.Down);
```

### Far-End Camera Control

```typescript
// Request control of another user's camera
await stream.requestFarEndCameraControl(userId);

// User receives request
client.on('far-end-camera-request-control', async (payload) => {
  const { userId, displayName } = payload;
  if (confirm(`${displayName} wants to control your camera. Allow?`)) {
    await stream.approveFarEndCameraControl(userId);
  } else {
    await stream.declineFarEndCameraControl(userId);
  }
});

// Control after approval
await stream.controlFarEndCamera(CameraControlCmd.ZoomIn, userId);
```

## Video Error Codes

| Code | Name | Description |
|------|------|-------------|
| 6100 | VIDEO_CAMERA_PERMISSION_DENIED | User denied camera access |
| 6101 | VIDEO_NO_CAMERA | No camera device detected |
| 6103 | VIDEO_CAMERA_TAKEN | Camera in use by another app |
| 6107 | VIDEO_VB_UNSUPPORTED | Virtual background not available |

## Best Practices

1. **Check camera availability** before starting video
2. **Request permissions** via user gesture (button click)
3. **Handle permission denial** gracefully with UI feedback
4. **Use appropriate quality** - 360p for gallery, 720p for speaker view
5. **Detach video** when stopping to free resources
6. **Listen for device changes** to handle camera disconnect
7. **Consider bandwidth** - lower quality for poor connections
