# Common Issues and Solutions

## Quick Diagnostic Checklist

When something isn't working, run through this checklist:

1. **SDK Lifecycle**: Did you follow `createClient() → init() → join() → getMediaStream()`?
2. **Stream Timing**: Did you call `getMediaStream()` AFTER `join()` completed?
3. **Event Listeners**: Are you listening for `peer-video-state-change`?
4. **attachVideo vs renderVideo**: Are you using `attachVideo()` (not deprecated `renderVideo()`)?
5. **Browser Permissions**: Did the user grant camera/microphone access?
6. **Browser Compatibility**: Is the browser supported (Chrome 80+, Firefox 75+, Safari 14+)?

---

## Most Common Issues

### 1. getMediaStream() Before join

**Symptom**: Confusion about whether `client.getMediaStream()` must wait for `join()`

**Cause**: The stream object can be created before `join()`, but most media operations still require the session to be joined first

**Solution**:
```javascript
// Allowed: create the stream object whenever you need it
const stream = client.getMediaStream();

// Recommended: join before calling media actions such as startVideo()
await client.join(...);
await stream.startVideo();
```

### 2. Video Not Displaying

**Symptom**: Video element created but shows black/nothing

**Causes**:
1. Not listening to `peer-video-state-change` event
2. Using deprecated `renderVideo()` instead of `attachVideo()`
3. Not appending returned element to DOM
4. Not setting width and height for the `video-player` custom element or its container
5. Container CSS causes SDK-inserted elements to render outside the expected tile

**Solution**:
```javascript
// 1. Use attachVideo(), not renderVideo()
const videoElement = await stream.attachVideo(userId, VideoQuality.Video_360P);

// 2. Append to DOM
container.appendChild(videoElement);

// 3. Listen for events
client.on('peer-video-state-change', async (payload) => {
  if (payload.action === 'Start') {
    const element = await stream.attachVideo(payload.userId, VideoQuality.Video_360P);
    container.appendChild(element);
  } else {
    await stream.detachVideo(payload.userId);
  }
});
```

```css
/* The custom video elements do not have a useful default size */
video-player-container {
  width: 100%;
  height: 1000px;
}

video-player {
  width: 100%;
  height: auto;
  aspect-ratio: 16 / 9;
}

video-player-container video-player,
video-player-container canvas,
video-player-container video {
  width: 100%;
  height: 100%;
  display: block;
}
```

### 3. Other Participants' Video Not Showing on Mid-Session Join

**Symptom**:
- Join mid-session, only your video shows, not others'
- The session appears to contain two copies of the same user after a reconnect

**Causes**:
1. Existing participants' videos don't auto-render
2. A disconnected user may remain in the session temporarily during failover detection, so others can still see the stale participant until the server removes them

**Solution**:
```javascript
// After joining, render any remote videos that are already active
async function renderExistingParticipants() {
  const container = document.querySelector('video-player-container');
  const currentUserId = client.getCurrentUserInfo().userId;

  for (const user of client.getAllUser()) {
    if (user.bVideoOn && user.userId !== currentUserId) {
      const element = await stream.attachVideo(user.userId, VideoQuality.Video_360P);
      container.appendChild(element);
    }
  }
}
```

```javascript
// Track reconnecting or stale participants and avoid treating them as active users
client.on('user-updated', (payload) => {
  payload.forEach((user) => {
    if (user.isInFailover) {
      // Show reconnecting state in the UI and avoid rendering this user as active
      stream.detachVideo(user.userId);
      markUserAsReconnecting(user.userId);
    } else {
      clearReconnectingState(user.userId);
    }
  });
});
```

If `isInFailover` is `true`, the participant is disconnected and may stay in the
session for up to about two minutes before the server removes them. Treat that
user as reconnecting in your UI instead of as a second active participant.

### 4. "ZoomVideo is not defined" or "WebVideoSDK is not defined"

**Symptom**: SDK global not available

**Causes**:
1. Network/ad blocker blocking `source.zoom.us` CDN
2. SDK script is injected dynamically or loaded asynchronously before access

**Solutions**:

**Solution 1 - Use a permitted fallback copy**:
```bash
# If your environment blocks `source.zoom.us`, you can mirror/self-host as a fallback
# only if permitted and you can keep versions in sync with the SDK you target.
curl "https://source.zoom.us/videosdk/zoom-video-2.4.0.min.js" -o public/js/zoom-video-sdk.min.js
```

```html
<script src="js/zoom-video-sdk.min.js"></script>
```

**Solution 2 - Wait for SDK to load when it is injected dynamically**:
```javascript
function waitForSDK(timeout = 10000) {
  return new Promise((resolve, reject) => {
    if (typeof WebVideoSDK !== 'undefined') {
      resolve();
      return;
    }
    const start = Date.now();
    const check = setInterval(() => {
      if (typeof WebVideoSDK !== 'undefined') {
        clearInterval(check);
        resolve();
      } else if (Date.now() - start > timeout) {
        clearInterval(check);
        reject(new Error('SDK failed to load'));
      }
    }, 100);
  });
}

await waitForSDK();
const ZoomVideo = WebVideoSDK.default;
```

### 5. CDN exports WebVideoSDK.default, not ZoomVideo

**Symptom**: `ZoomVideo.createClient()` fails with CDN

**Cause**: CDN exports as `WebVideoSDK`, not `ZoomVideo`

**Solution**:
```javascript
// NPM
import ZoomVideo from '@zoom/videosdk';

// CDN
const ZoomVideo = WebVideoSDK.default;  // Note: .default!

const client = ZoomVideo.createClient();
```

### 6. Join Fails with "Invalid signature"

**Symptom**: `join()` throws error about invalid signature

**Causes**:
1. JWT expired (check `exp` claim)
2. JWT malformed
3. Wrong SDK key/secret
4. Topic doesn't match JWT `tpc` claim

**Solution**:
1. Generate JWT on server side
2. Check JWT expiration (typically 24h)
3. Verify topic matches JWT `tpc` value
4. Verify SDK key is correct

### 7. Camera/Microphone Permission Denied

**Symptom**: `startVideo()` or `startAudio()` fails

**Cause**: Browser permission denied

**Solution**:
```javascript
// Check permission-related errorCode values instead of relying on error.type
try {
  await stream.startVideo();
} catch (error) {
  if (error.errorCode === ExceptionCode.VIDEO_CAMERA_PERMISSION_DENIED) {
    alert('Please allow camera access in browser settings');
  }
}

try {
  await stream.startAudio();
} catch (error) {
  if (error.errorCode === ExceptionCode.AUDIO_CAPTURE_FAILED) {
    alert('Please allow microphone access in browser settings');
  }
}
```

### 8. HD Video Not Working

**Symptom**: Video quality stays at 360p despite `{ hd: true }`

**Causes**:
1. Network conditions are not good enough for 720p
2. The camera or device cannot reliably capture HD video
3. The account, CPU load, or current session conditions do not support HD video
4. SharedArrayBuffer or cross-origin isolation settings may affect some advanced browser capabilities, but they are usually not the primary reason 720p is unavailable

**Solution**:
```javascript
// Check whether the current account and session conditions support HD
if (stream.isSupportHDVideo()) {
  await stream.startVideo({ hd: true });
} else {
  console.warn('720p is not available in the current environment');
  await stream.startVideo();
}

// Check how many higher-quality videos the current environment can render
console.log(stream.getMaxRenderableVideos());
```

Use `stream.isSupportHDVideo()` to decide whether to request 720p, and treat
network quality and camera/device capability as the first things to verify.
If you need to confirm what resolution is actually being received, subscribe to
video statistics and inspect the height value.

```javascript
stream.subscribeVideoStatisticData();

client.on('video-statistic-data-change', (payload) => {
  console.log(payload.height); // 360, 720, or 1080
});
```

If you are specifically debugging browser isolation issues, then also verify
your SharedArrayBuffer-related headers:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

### 9. Screen Share Element Type Error

**Symptom**: `startShareScreen()` fails or shows nothing

**Cause**: Using wrong element type (video vs canvas)

**Solution**:
```javascript
// Check which element type to use
if (stream.isStartShareScreenWithVideoElement()) {
  const video = document.getElementById('share-video');
  await stream.startShareScreen(video as unknown as HTMLCanvasElement);
} else {
  const canvas = document.getElementById('share-canvas');
  await stream.startShareScreen(canvas);
}
```

### 10. CORS Error to log-external-gateway.zoom.us

**Symptom**: Console shows CORS errors to Zoom telemetry

**Cause**: COOP/COEP headers blocking telemetry

**Impact**: None - harmless. SDK works fine.

**Solution**: Ignore these errors. They're telemetry-related and don't affect functionality.

### 11. Video Stops Rendering After Navigating / Toggling Layout

**Symptom**:
- Video renders fine at first, then goes black or stops rendering after switching
  routes/tabs, opening and closing the video view several times, or toggling a
  layout repeatedly.
- May surface as `active-media-failed` with `WebGlContextInvalid`, or console
  warnings about too many active WebGL contexts.

**Cause**: The `video-player-container` is being unmounted and recreated (e.g.
conditional rendering in React/Vue, route changes, or recreating the element on
every layout change). The container holds the shared rendering surface for the
videos under it, and the browser caps how many such surfaces can exist.
Destroy/recreate cycles churn these surfaces and eventually exhaust the limit.

**Solution**:
- Mount `video-player-container` once and keep it for the session lifetime.
- Do not place it behind conditional rendering that destroys it; keep it mounted
  and toggle visibility with CSS (`display: none` / `visibility: hidden`).
- Individual `video-player` elements can still be freely added/removed via
  `attachVideo()` / `detachVideo()` — only the container must stay stable.

```javascript
// Anti-pattern: container destroyed whenever `inSession` flips
// {inSession && <video-player-container />}

// Better: container always mounted, hidden with CSS when not in use
// <video-player-container style={{ display: inSession ? 'flex' : 'none' }} />
```

---

## Error Handling Reference

Use `error.errorCode` for precise troubleshooting, and use `error.type` only as
a coarse category. See [../references/error-codes.md](../references/error-codes.md)
for the full `ExceptionCode` and `ActiveMediaFailedCode` reference.

Common examples:

| Scenario | Check |
|----------|-------|
| Camera permission denied | `ExceptionCode.VIDEO_CAMERA_PERMISSION_DENIED` |
| Microphone capture or permission failure | `ExceptionCode.AUDIO_CAPTURE_FAILED` |
| API called before join completes | `ExceptionCode.STREAM_SESSION_JOIN_REQUIRED` |
| Wrong render element type | `ExceptionCode.STREAM_MISMATCH_RENDER_ELEMENT` |
| Duplicate join call | `ExceptionCode.CLIENT_DUPLICATED_JOIN` |

---

## Browser-Specific Issues

For the current feature matrix, use [../references/browser-support.md](../references/browser-support.md).

Common browser-specific checks:

| Issue | Check |
|-------|-------|
| Mobile users cannot start screen share | Expected: iOS/iPadOS and Android browsers can receive screen share but cannot send it |
| Firefox WebRTC video behaves differently | Expected: WebRTC video is not currently listed as supported on Firefox |
| Virtual background fails on Firefox or Safari | Check whether the app is using WebAssembly video and whether `SharedArrayBuffer` / cross-origin isolation is available |
| 1080p does not work outside Chrome/Edge | Expected: 1080p send/receive is currently listed only for Chrome and Edge |
| Edge shows a purple background while receiving video | Check Edge's "Enhance your security on the web" strict mode or add the site to an exception list |
| Mobile browser support differs by browser name on iOS | Expected: all iOS/iPadOS browsers use WebKit, so support is OS-version based |

---

## Debugging Tips

### 1. Capture the failing API result

```javascript
try {
  await stream.startVideo();
} catch (error) {
  console.table({
    type: error.type,
    reason: error.reason,
    errorCode: error.errorCode,
  });
}
```

Use `error.errorCode` first, then look up the code in [../references/error-codes.md](../references/error-codes.md).

### 2. Trace session and participant state

```javascript
client.on('connection-change', (payload) => {
  console.log('[connection-change]', payload);
});

client.on('user-added', (payload) => {
  console.log('[user-added]', payload);
});

client.on('user-updated', (payload) => {
  console.log('[user-updated]', payload);
});

client.on('user-removed', (payload) => {
  console.log('[user-removed]', payload);
});

client.on('peer-video-state-change', (payload) => {
  console.log('[peer-video-state-change]', payload);
});
```

For stale users or reconnect issues, inspect `isInFailover` from `user-updated`.

### 3. Capture media failures that happen after media starts

```javascript
client.on('active-media-failed', (payload) => {
  console.log('[active-media-failed]', payload.code, payload.message);
});
```

This catches cases like camera/microphone permission reset, interrupted streams,
WebGL issues, and WebAssembly out-of-memory events.

### 4. Collect quality data only when needed

```javascript
stream.subscribeVideoStatisticData();
client.on('video-statistic-data-change', (payload) => {
  console.log('[video-statistic-data-change]', payload);
});

stream.subscribeAudioStatisticData();
client.on('audio-statistic-data-change', (payload) => {
  console.log('[audio-statistic-data-change]', payload);
});
```

Use this for network, FPS, resolution, packet loss, or HD quality issues. For
layout problems, inspect the DOM and CSS first.

### 5. Prepare a Zoom-investigable report

Do not expect application developers to interpret raw SDK internal logs. For
reproducible connection, crash, or media quality issues, keep detailed telemetry
enabled and include a tracking ID that Zoom can use to find the client report.

```javascript
await client.init('en-US', 'Global', {
  // true by default; keep it enabled unless your privacy requirements differ
  isLogDetailed: true,
});
```

Add `telemetry_tracking_id` to the Video SDK JWT payload. Use a unique UUID per
session or user report, then include that ID when escalating the issue to Zoom.

```json
{
  "telemetry_tracking_id": "a8b7f844-1d32-4eeb-93a4-785a77f49428"
}
```

If you collect post-session quality feedback, send it through
`client.getLoggerClient().reportRating(score, feedback)` so low-quality sessions
can be correlated with telemetry.

---

## Real-World Integration Pitfalls (Custom Waiting Room Flows)

These are specific to custom flows that move users from a waiting session into a main session.

### A) Transfer works, but customer remote video never appears

**Symptom**: Customer reaches main session but does not see advisor video.

**Likely causes**:
1. Advisor is not publishing video (`bVideoOn` is false)
2. Event listeners were registered against the previous waiting session state
3. The main-session render pass only waits for future events and never renders users already in the session

**Fix pattern**:
- Bind listeners once and gate logic by current session mode.
- After joining the main session, immediately run `client.getAllUser()` and attach any users with `bVideoOn`.
- Handle `peer-video-state-change`, `user-added`, and `user-updated` for subsequent state changes.
- Skip users marked `isInFailover`; show a reconnecting state instead of rendering them as active.

### B) Command channel transfer message is missed

**Symptom**: Admit clicked, but customer does not transfer.

**Cause**: Command channel is session-scoped and does not replay history. It is available after `client.join()` and `client.getCommandClient()`. If the customer is not fully joined to the waiting session, or if the app expects the command to survive a session transition, the message can be missed.

**Fix**:
- Treat command channel as a best-effort in-session signal, not durable transfer state.
- Keep backend transfer state and let the customer fetch transfer details after joining the waiting session.
- Initialize command channel only after join, then register `command-channel-message` handlers.
- Add a one-time transfer-state lookup on customer waiting-session join as the race guard.

---

## Related Documentation

- [SKILL.md](../SKILL.md) - Quick reference
