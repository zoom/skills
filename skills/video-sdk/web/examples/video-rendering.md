# Video Rendering

Complete guide to rendering video using `attachVideo()` in the Zoom Video SDK for Web.

## Critical Rule

**NEVER use `renderVideo()` - it's deprecated. Always use `attachVideo()`.**

## VideoQuality Enum

```typescript
import { VideoQuality } from '@zoom/videosdk';

// Available quality levels (value = numeric enum)
VideoQuality.Video_90P   // 0 - Thumbnail
VideoQuality.Video_180P  // 1 - Low quality
VideoQuality.Video_360P  // 2 - Standard (recommended default)
VideoQuality.Video_720P  // 3 - HD
VideoQuality.Video_1080P // 4 - Full HD (requires webrtc mode)
```

## Basic Video Rendering

### DOM Surface Rule for `attachVideo()`

`attachVideo()` returns a SDK-managed `video-player` custom element. Make that returned element the visible tile surface.

When migrating from older `renderVideo(canvas, ...)` layouts, remove the old canvas, avatar, name label, and absolute-positioned tile overlay from the video area. Do not fix black or hidden video with `z-index`, transparent backgrounds, or pointer-event changes; that usually leaves a competing DOM layer over the SDK video.

Use a normal-flow `video-player-container` grid and append only returned `video-player` elements:

```css
video-player-container {
  width: 100%;
  display: grid !important;
  grid-template-columns: repeat(1, minmax(0, 1fr));
  gap: 10px;
}

video-player-container:has(> :nth-child(2)) {
  grid-template-columns: repeat(2, minmax(0, 1fr));
}

video-player-container:has(> :nth-child(5)) {
  grid-template-columns: repeat(3, minmax(0, 1fr));
}

video-player {
  width: 100%;
  height: auto;
  aspect-ratio: 16 / 9;
  display: block;
}
```

### Start and Render Own Video

```javascript
import ZoomVideo, { VideoQuality } from '@zoom/videosdk';

const client = ZoomVideo.createClient();
let stream;

async function startOwnVideo() {
  // Ensure you're joined first
  stream = client.getMediaStream();
  
  // Step 1: Start capturing video
  await stream.startVideo();
  
  // Step 2: Get current user ID
  const currentUser = client.getCurrentUserInfo();
  
  // Step 3: Attach video to DOM
  const videoElement = await stream.attachVideo(
    currentUser.userId,
    VideoQuality.Video_360P
  );
  
  // Step 4: Add to container
  document.getElementById('my-video-container').appendChild(videoElement);
}
```

### Render Remote Participant Video

```javascript
// Listen for remote video state changes
client.on('peer-video-state-change', async (payload) => {
  const { action, userId } = payload;
  
  if (action === 'Start') {
    // Remote user started video - render it
    const element = await stream.attachVideo(userId, VideoQuality.Video_360P);
    
    // Add to their video container
    const container = document.getElementById(`video-${userId}`);
    if (container) {
      container.appendChild(element);
    }
  } else if (action === 'Stop') {
    // Remote user stopped video - remove element
    await stream.detachVideo(userId);
    
    // Clean up DOM
    const container = document.getElementById(`video-${userId}`);
    if (container) {
      container.innerHTML = '';
    }
  }
});
```

## Mid-Session Join: Rendering Existing Participants

When you join a session that already has participants with video on, you won't receive `peer-video-state-change` events for them. You must manually render their videos.

```javascript
async function renderExistingParticipants() {
  // Wait a moment for participant list to populate
  await new Promise(resolve => setTimeout(resolve, 500));
  
  const participants = client.getAllUser();
  const currentUserId = client.getCurrentUserInfo().userId;
  
  for (const participant of participants) {
    // Skip self
    if (participant.userId === currentUserId) continue;
    
    // Check if they have video on
    if (participant.bVideoOn) {
      const element = await stream.attachVideo(
        participant.userId,
        VideoQuality.Video_360P
      );
      
      const container = document.getElementById(`video-${participant.userId}`);
      if (container) {
        container.appendChild(element);
      }
    }
  }
}

// Call after joining and starting your own video
await startOwnVideo();
await renderExistingParticipants();
```

## Participant Media Reconciliation

The SDK does not automatically subscribe/render every participant for custom UIs. Treat `client.getAllUser()` as the source of truth and reconcile the DOM after join and after participant/video events. This covers both users who were already in the session and users who join or toggle video later.

```javascript
const renderedUsers = new Map();
const renderingUsers = new Map();

async function reconcileVideoTiles() {
  const users = client.getAllUser();
  const selfId = client.getCurrentUserInfo().userId;
  const usersWithVideo = new Map(
    users
      .filter((user) => user.userId !== selfId && user.bVideoOn)
      .map((user) => [user.userId, user])
  );

  for (const [userId, element] of renderedUsers) {
    if (!usersWithVideo.has(userId)) {
      const detached = await stream.detachVideo(userId, element);
      const elements = Array.isArray(detached) ? detached : [detached];
      elements.filter(Boolean).forEach((element) => element.remove());
      renderedUsers.delete(userId);
    }
  }

  for (const [userId, user] of usersWithVideo) {
    if (!renderedUsers.has(userId) && !renderingUsers.has(userId)) {
      const renderPromise = (async () => {
        if (renderedUsers.has(userId)) return;

        const player = await stream.attachVideo(userId, VideoQuality.Video_360P);

        const stillWanted = client.getAllUser().some((latestUser) =>
          latestUser.userId === userId &&
          latestUser.userId !== client.getCurrentUserInfo().userId &&
          latestUser.bVideoOn
        );
        if (!stillWanted || renderedUsers.has(userId)) {
          await stream.detachVideo(userId, player).catch(console.warn);
          player.remove();
          return;
        }

        player.setAttribute('data-user-id', String(userId));
        player.setAttribute('data-user-name', user.displayName || String(userId));
        document.querySelector('video-player-container').appendChild(player);
        renderedUsers.set(userId, player);
      })();

      renderingUsers.set(userId, renderPromise);
      renderPromise.finally(() => {
        if (renderingUsers.get(userId) === renderPromise) {
          renderingUsers.delete(userId);
        }
      });
    }
  }
}

await reconcileVideoTiles();
setTimeout(reconcileVideoTiles, 1000);
setTimeout(reconcileVideoTiles, 3000);

client.on('user-added', reconcileVideoTiles);
client.on('user-updated', reconcileVideoTiles);
client.on('user-removed', reconcileVideoTiles);
client.on('peer-video-state-change', reconcileVideoTiles);
```

For 1:M galleries with a dedicated local preview, keep self rendering separate from remote rendering:

- Render the local camera in a dedicated self container after `stream.startVideo()` by attaching `client.getCurrentUserInfo().userId` there.
- Exclude `client.getCurrentUserInfo().userId` from the remote participant gallery. Rendering self again as a remote tile creates duplicate local video.
- Track both rendered users and in-flight `attachVideo()` promises by `userId`. Initial join reconciliation, delayed reconciliation, and SDK events can overlap; without an in-flight guard, the same remote user can be appended twice.
- Detach with the specific returned `video-player` element when available: `stream.detachVideo(userId, element)`.

## Quality Selection Strategy

```javascript
// Determine quality based on use case
function getQualityForLayout(totalParticipants, isSpotlight = false) {
  if (isSpotlight) {
    // Spotlighted/active speaker - highest quality
    return VideoQuality.Video_720P;
  }
  
  if (totalParticipants <= 4) {
    // Small meeting - good quality for all
    return VideoQuality.Video_360P;
  }
  
  if (totalParticipants <= 9) {
    // Medium meeting - balanced quality
    return VideoQuality.Video_180P;
  }
  
  // Large meeting - thumbnails
  return VideoQuality.Video_90P;
}

// Dynamic quality adjustment
async function updateVideoQualities() {
  const participants = client.getAllUser().filter(p => p.bVideoOn);
  const quality = getQualityForLayout(participants.length);
  
  for (const participant of participants) {
    // Re-attach with new quality
    await stream.detachVideo(participant.userId);
    const element = await stream.attachVideo(participant.userId, quality);
    
    const container = document.getElementById(`video-${participant.userId}`);
    if (container) {
      container.innerHTML = '';
      container.appendChild(element);
    }
  }
}
```

## HD Video (720P/1080P)

To use HD video, enable WebRTC mode during initialization:

```javascript
await client.init('en-US', 'Global', {
  patchJsMedia: true,
  webrtc: true,  // Required for HD video
});

// Now you can use higher qualities
const element = await stream.attachVideo(userId, VideoQuality.Video_720P);

// Check if HD is supported on this device
if (stream.isSupportHDVideo()) {
  const hdElement = await stream.attachVideo(userId, VideoQuality.Video_1080P);
}
```

## Multiple Video Rendering

Check device capability for rendering multiple videos:

```javascript
// Check max renderable videos
const maxVideos = stream.getMaxRenderableVideos();
console.log('Can render up to', maxVideos, 'videos');

// Check if multiple video rendering is supported
if (stream.isSupportMultipleVideos()) {
  // Can render multiple participant videos
} else {
  // Limited to fewer simultaneous videos
  // Consider using active speaker mode
}
```

## Detaching Video

```javascript
// Detach specific user's video
const elements = await stream.detachVideo(userId);

// elements can be a single element or array
if (Array.isArray(elements)) {
  elements.forEach(el => el.remove());
} else {
  elements.remove();
}

// Detach from specific element
const specificElement = document.querySelector(`#video-${userId} video-player`);
await stream.detachVideo(userId, specificElement);
```

## Mirror Self Video

```javascript
// Mirror your own video (selfie mode)
await stream.mirrorVideo(true);

// Check current mirror state
const isMirrored = stream.isVideoMirrored();
```

## Stop Video

```javascript
// Stop capturing video (turns off camera)
await stream.stopVideo();

// The attached video element will show black/placeholder
// Detach to clean up
const currentUser = client.getCurrentUserInfo();
await stream.detachVideo(currentUser.userId);
```

## React MediaStream Pattern

This is the direct `MediaStream` sample pattern. It follows the official React sample's attachment model without switching to the `@zoom/videosdk-react` wrapper package.

```tsx
function MediaStreamVideoGrid({ client, stream }) {
  const containerRef = React.useRef<HTMLElement | null>(null);
  const [users, setUsers] = React.useState(() => client.getAllUser());

  React.useEffect(() => {
    const refresh = () => setUsers(client.getAllUser());
    refresh();
    client.on('user-added', refresh);
    client.on('user-removed', refresh);
    client.on('user-updated', refresh);
    client.on('peer-video-state-change', refresh);
    client.on('video-capturing-change', refresh);
    return () => {
      client.off('user-added', refresh);
      client.off('user-removed', refresh);
      client.off('user-updated', refresh);
      client.off('peer-video-state-change', refresh);
      client.off('video-capturing-change', refresh);
    };
  }, [client]);

  React.useEffect(() => {
    const container = containerRef.current;
    if (!container || !stream) return;

    users.forEach(async (user) => {
      const selector = `video-player[data-user-id="${user.userId}"]`;

      if (user.bVideoOn && !container.querySelector(selector)) {
        const player = await stream.attachVideo(user.userId, VideoQuality.Video_360P);
        player.setAttribute('data-user-id', String(user.userId));
        container.appendChild(player);
      }

      if (!user.bVideoOn && container.querySelector(selector)) {
        const detached = await stream.detachVideo(user.userId);
        const players = Array.isArray(detached) ? detached : [detached];
        players.filter(Boolean).forEach((player) => player.remove());
        container.querySelectorAll(selector).forEach((player) => player.remove());
      }
    });
  }, [stream, users]);

  return React.createElement('video-player-container', { ref: containerRef });
}
```

## Complete React Component

```typescript
import React, { useEffect, useRef, useState } from 'react';
import ZoomVideo, { VideoClient, Stream, VideoQuality } from '@zoom/videosdk';

interface VideoTileProps {
  userId: number;
  stream: typeof Stream;
  quality?: VideoQuality;
}

export const VideoTile: React.FC<VideoTileProps> = ({ 
  userId, 
  stream, 
  quality = VideoQuality.Video_360P 
}) => {
  const containerRef = useRef<HTMLDivElement>(null);
  const [isAttached, setIsAttached] = useState(false);

  useEffect(() => {
    let mounted = true;

    const attachVideo = async () => {
      if (!containerRef.current || !stream) return;

      try {
        const element = await stream.attachVideo(userId, quality);
        if (mounted && containerRef.current) {
          containerRef.current.innerHTML = '';
          containerRef.current.appendChild(element);
          setIsAttached(true);
        }
      } catch (error) {
        console.error('Failed to attach video:', error);
      }
    };

    attachVideo();

    return () => {
      mounted = false;
      if (isAttached) {
        stream.detachVideo(userId).catch(console.error);
      }
    };
  }, [userId, stream, quality, isAttached]);

  return (
    <div 
      ref={containerRef} 
      className="video-tile"
      style={{ width: '100%', height: '100%', background: '#1a1a1a' }}
    />
  );
};

// Usage in parent component
export const VideoGrid: React.FC<{ client: typeof VideoClient; stream: typeof Stream }> = ({ 
  client, 
  stream 
}) => {
  const [participants, setParticipants] = useState<any[]>([]);

  useEffect(() => {
    // Initial load
    setParticipants(client.getAllUser().filter(p => p.bVideoOn));

    // Listen for changes
    const handleVideoChange = async (payload: { action: string; userId: number }) => {
      if (payload.action === 'Start') {
        setParticipants(prev => {
          const user = client.getUser(payload.userId);
          if (user && !prev.find(p => p.userId === payload.userId)) {
            return [...prev, user];
          }
          return prev;
        });
      } else {
        setParticipants(prev => prev.filter(p => p.userId !== payload.userId));
      }
    };

    client.on('peer-video-state-change', handleVideoChange);

    return () => {
      client.off('peer-video-state-change', handleVideoChange);
    };
  }, [client]);

  const quality = participants.length <= 4 
    ? VideoQuality.Video_360P 
    : VideoQuality.Video_180P;

  return (
    <div className="video-grid">
      {participants.map(p => (
        <VideoTile 
          key={p.userId} 
          userId={p.userId} 
          stream={stream} 
          quality={quality}
        />
      ))}
    </div>
  );
};
```

## Error Handling

```javascript
async function safeAttachVideo(userId, quality) {
  try {
    const element = await stream.attachVideo(userId, quality);
    return element;
  } catch (error) {
    console.error('attachVideo failed:', error);
    
    if (error.type === 'INVALID_OPERATION') {
      // User may have stopped video, or not in session
      console.log('User video not available');
    } else if (error.type === 'INTERNAL_ERROR') {
      // SDK internal error - may need to retry
      await new Promise(r => setTimeout(r, 1000));
      return stream.attachVideo(userId, quality);
    }
    
    return null;
  }
}
```

## Key Points

1. **Use `attachVideo()`, NOT `renderVideo()`** - `renderVideo()` is deprecated
2. **Listen to `peer-video-state-change`** - Required for remote video
3. **Handle mid-session join** - Manually reconcile `client.getAllUser()` and `bVideoOn`
4. **Detach before re-attaching** - When changing quality
5. **Check device capabilities** - `getMaxRenderableVideos()`, `isSupportMultipleVideos()`
6. **Enable WebRTC for HD** - Required for 720P/1080P

## Related Documentation

- [Session Join](session-join-pattern.md) - Initial setup
- [Event Handling](event-handling.md) - All video events
- [Common Issues](../troubleshooting/common-issues.md) - Troubleshooting
