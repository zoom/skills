# React Implementation Guide

Complete React 19(latest) + Vite 8(latest) + TypeScript 6(latest) + shadcn/ui implementation for Zoom Video SDK.

## Features

This implementation includes:

- ✅ **Audio/Video Controls** with loading states and proper icon indicators
- ✅ **Screen Sharing** with full viewing support and WebCodecs compatibility
- ✅ **Sharing Bar** with elegant notification when user is screen sharing
- ✅ **Real-time Participant List** with auto-updates using event payloads
- ✅ **Chat System** with message history
- ✅ **Active Speaker Detection** with visual indicators
- ✅ **Gallery & Speaker View** modes
- ✅ **URL Parameter Support** for direct session joining and sharing
- ✅ **Session Information Dialog** with connection details
- ✅ **Responsive Design** using shadcn/ui components
- ✅ **Promise-based State Management** for accurate button states
- ✅ **Professional Icons** from lucide-react (no emojis)
- ✅ **fetchToken** get token from jwt server use post

## Prerequisites

- **Bun** (recommended) or npm/yarn
- **Node.js** 24+
- Basic knowledge of React hooks and TypeScript

## Node.js(jsrsasign) Quick Start JWT token Generator Service:

```bash
git clone https://github.com/zoom/videosdk-auth-endpoint-sample.git

cd videosdk-auth-endpoint-sample

bun install
# or 
npm install
# Rename .env.example to .env, edit the file contents to include your Zoom Video SDK key and secret, save the file contents, and close the file
mv .env.example .env
# Start the server
# server code https://raw.githubusercontent.com/zoom/videosdk-auth-endpoint-sample/refs/heads/master/src/index.js
bun run start
# call 
# or
npm run start
```

### fetchToken test
```bash
# POST role + sessionName to your token server
curl -s -X POST "http://localhost:4000/" \
  -H "Content-Type: application/json" \
  -d '{"role":1,"sessionName":"example-session"}'

# Example response:
# {
#   "signature": "XXXXXXXXXXXX"
# }
```

## Project Setup

### 1. Create Vite Project

```bash
# Using Bun (recommended)
bun create vite@latest my-zoom-app-react --template react-ts
cd my-zoom-app-react

# Or using npm
npm create vite@latest my-zoom-app-react -- --template react-ts
cd my-zoom-app-react
```

### 2. Install Dependencies

```bash
# Core dependencies
bun add @zoom/videosdk@latest

# Development tools
bun add -D @biomejs/biome@latest

# UI components (shadcn/ui)
bunx shadcn@latest init
bunx shadcn@latest add button card avatar badge dialog input scroll-area

# Initialize Biome for linting/formatting
bunx biome init
```

### 3. Additional UI Icons

```bash
# Install lucide-react for icons (if not already installed by shadcn)
bun add lucide-react@latest
```

## CRITICAL: Vite Configuration

**You MUST configure Vite with COOP/COEP headers** for SharedArrayBuffer support:

```typescript
// vite.config.ts
import tailwindcss from "@tailwindcss/vite";
import react from "@vitejs/plugin-react";
import path from "node:path";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  server: {
    headers: {
      // required for SharedArrayBuffer for gallery view if not use video_webrtc_mode
      "Cross-Origin-Opener-Policy": "same-origin",
      "Cross-Origin-Embedder-Policy": "require-corp",
    },
  },
});
```

## CRITICAL: Custom Element CSS

`<video-player-container>` and `<video-player>` are custom elements and default to `display: inline` with **zero size**. Without explicit CSS, `startVideo()` and `attachVideo()` succeed silently but no tile is visible (symptom: black area, no console error, camera permission granted, participant shows `bVideoOn=true`).

Add these rules to your global stylesheet (e.g. `src/index.css`):

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
  border-radius: 0.5rem;
  overflow: hidden;
}
```

Tailwind utilities on the container (`w-full h-full grid`) are not a reliable substitute — custom elements need the base `display` rule applied globally so the SDK's children size correctly inside the grid.

**Gallery-view grid template:** Do **not** use Tailwind arbitrary values like `grid-cols-[repeat(auto-fit,minmax(260px,1fr))]` on `<video-player-container>` for the gallery layout. The comma-in-bracket arbitrary value is fragile under Tailwind v4's scanner and, even when it compiles, the global `video-player-container { display: grid }` rule can land after class rules in the cascade and leave you with a single column (symptom: gallery toggle "does nothing"). Set `gridTemplateColumns` inline instead:

```tsx
<video-player-container
  style={{
    display: "grid",
    width: "100%",
    height: "100%",
    gap: "0.5rem",
    gridTemplateColumns:
      viewMode === "gallery"
        ? "repeat(auto-fit, minmax(260px, 1fr))"
        : "1fr",
    gridAutoRows: viewMode === "gallery" ? "1fr" : "auto",
  }}
/>
```

Inline styles win over the global element selector and are immune to Tailwind scanner edge cases.

## CRITICAL: TypeScript Custom Elements

```typescript
// src/types/zoom-elements.d.ts
import 'react'

declare module 'react' {
  namespace JSX {
    interface IntrinsicElements {
      'video-player-container': React.DetailedHTMLProps<
        React.HTMLAttributes<HTMLElement>,
        HTMLElement
      >
      'video-player': React.DetailedHTMLProps<
        React.HTMLAttributes<HTMLElement>,
        HTMLElement
      >
    }
  }
}
```

## Project Structure

```
src/
├── components/
│   ├── video-session/
│   │   ├── SessionStatus.tsx
│   │   ├── VideoGrid.tsx
│   │   ├── ParticipantList.tsx
│   │   ├── ChatPanel.tsx
│   │   ├── ControlBar.tsx
│   │   ├── SessionInfo.tsx
│   │   ├── SharingBar.tsx
│   │   └── JoinForm.tsx
│   └── ui/
├── hooks/
│   ├── useZoomClient.ts
│   ├── useSessionStatus.ts
│   ├── useMediaStream.ts
│   ├── useParticipants.ts
│   ├── useChat.ts
│   └── useActiveSpeaker.ts
├── types/
│   ├── zoom.ts
│   └── zoom-elements.d.ts
└── App.tsx
```

## Audio/Video Loading States

The media controls include loading states for better user feedback:

### Implementation

**useMediaStream Hook:**
- Tracks `isAudioLoading` and `isVideoLoading` states
- Updates states using async/await for sequential operations
- Ensures UI updates only after SDK operations complete
- Handles errors gracefully without breaking UI state

**Audio Button States:**
- **Not Joined** (`!audioOn`): Shows Headphones icon - "Join Computer Audio"
- **Joined & Muted** (`audioOn && muted`): Shows MicOff icon - "Unmute"
- **Joined & Unmuted** (`audioOn && !muted`): Shows Mic icon - "Mute"
- **Loading** (`isAudioLoading`): Shows Loader2 spinner, button disabled

**Video Button States:**
- **Off** (`!videoOn`): Shows VideoOff icon - "Start Video"
- **On** (`videoOn`): Shows Video icon - "Stop Video"
- **Loading** (`isVideoLoading`): Shows Loader2 spinner, button disabled

### Key Benefits

1. **Accurate State Tracking**: UI reflects actual SDK state, not premature assumptions
2. **User Feedback**: Loading spinners prevent confusion and duplicate clicks
3. **Error Resilience**: Failed operations don't break button state
4. **Graceful Mute Handling**: If mute fails after audio start, shows unmuted state correctly

## URL Parameter Feature

The application supports URL parameters for seamless joining and sharing:

### Auto-Join with URL Parameters

Users can join sessions directly via URL without seeing the join form:

```
http://localhost:5173/?sessionName=my-session&userName=John&token=eyJhbGci...
```

**Required Parameters:**
- `sessionName` - The Zoom session name to join
- `userName` - Display name for the user
- `token` - JWT token for authentication

**Behavior:**
1. All three parameters present → Auto-join after 500ms delay (ensures client is ready)
2. Partial parameters → Pre-populate form fields, show form
3. No parameters → Show form with defaults

### URL State Management

**After Joining:**
- URL is automatically updated with session parameters
- Enables sharing the current session URL with others
- Parameters are added using `window.history.pushState()`

**After Leaving:**
- URL parameters are automatically removed
- Clean URL state for next session
- Parameters are removed using `window.history.pushState()`

### Implementation Notes

**CRITICAL Timing Considerations:**
- 500ms delay before auto-join ensures Zoom client is fully initialized
- Prevents "OPERATION_CANCELLED" / "LEAVING_MEETING" errors
- Uses `useRef` to prevent multiple auto-join attempts
- Empty dependency array in client initialization useEffect prevents re-renders

**Example Usage:**
```typescript
// Generate shareable URL after joining
const shareUrl = `${window.location.origin}/?sessionName=${sessionName}&userName=${userName}&token=${token}`;

// User can share this URL for instant session access
```

## Screen Sharing Feature

The application includes comprehensive screen sharing with both sharing and viewing capabilities.

### Screen Share Viewing

**Full-Screen Share View:**
- When a participant shares their screen, the entire interface switches to display only the shared screen
- Video grid is automatically hidden during screen sharing
- Large, centered display optimized for viewing shared content
- Screen share indicator shows who is sharing with `MonitorPlay` icon

**Automatic State Management:**
- Tracks `activeShareUserId` to identify who is currently sharing
- Automatically switches between video grid and share view
- Restores video grid when screen sharing stops
- Handles multiple share state events for reliable tracking

### Implementation Architecture

**Event Handling:**
```typescript
// Primary events for screen sharing
client.on('active-share-change', shareHandler)       // Own sharing state
client.on('peer-share-state-change', peerShareHandler) // Peer sharing state
client.on('passively-stop-share', passivelyStopShareHandler) // Forced stop
client.on('share-content-change', shareContentChangeHandler) // Content updates
client.on('share-content-dimension-change', dimensionHandler) // Size changes
```

**State Tracking:**
- `sharing`: Boolean tracking if current user is sharing
- `activeShareUserId`: Number tracking which participant is currently sharing
- Both current user and peer sharing states are tracked independently

**WebCodecs Compatibility:**
- Uses `<video>` element instead of `<canvas>` for better WebCodecs support
- Hidden video element created on stream initialization
- Automatic cleanup on component unmount
- Proper muted and autoplay attributes for seamless rendering

### ShareView Component

**Purpose:** Dedicated component for rendering shared screen content

**Key Features:**
- Uses `attachShareView(userId)` for content rendering
- Full-screen layout with centered content
- Object-fit: contain for proper aspect ratio
- Visual indicator showing sharer's name
- Automatic cleanup with `detachShareView(userId)`

**Usage in VideoGrid:**
```typescript
// Prioritizes share view over video grid
if (sharingParticipant && stream) {
  return (
    <ShareView
      userId={sharingParticipant.userId}
      stream={stream}
      userName={sharingParticipant.displayName}
    />
  )
}
```

### SharingBar Component

**Purpose:** Elegant notification bar for current user when screen sharing

**Key Features:**
- Compact, pill-shaped design with gradient background
- Positioned at top-center with smooth fade-in/slide-in animation
- Pulsing indicator dot for active sharing status
- Quick "Stop" button for immediate share termination
- No video preview (SDK limitation - can't attach same share view twice)

**Implementation:**
```typescript
// SharingBar.tsx - Minimal, elegant design
export function SharingBar({ onStopSharing }: SharingBarProps) {
  return (
    <div className="fixed top-4 left-1/2 -translate-x-1/2 z-50 animate-in fade-in slide-in-from-top-2 duration-300">
      <div className="bg-gradient-to-r from-green-600 to-green-500 text-white rounded-full shadow-xl flex items-center gap-3 px-5 py-2.5">
        <div className="flex items-center gap-2.5">
          <div className="relative">
            <MonitorPlay className="h-4 w-4" />
            <span className="absolute -top-1 -right-1 h-2 w-2 bg-white rounded-full animate-pulse" />
          </div>
          <span className="font-medium text-sm">You're sharing your screen</span>
        </div>
        <Button variant="secondary" size="sm" onClick={onStopSharing}>
          <MonitorOff className="h-3.5 w-3.5 mr-1.5" />
          Stop
        </Button>
      </div>
    </div>
  )
}
```

**Usage in SessionPage:**
```typescript
{/* Show sharing bar when current user is sharing */}
{sharing && (
  <SharingBar onStopSharing={handleToggleScreenShare} />
)}
```

**Design Principles:**
- **Minimalist**: 70% smaller than traditional sharing bars
- **Non-intrusive**: Small footprint doesn't block video content
- **Animated**: Smooth transitions for professional feel
- **Accessible**: Clear visual indicator and immediate action button

### User Experience

**For Sharer:**
1. Click screen share button in control bar
2. Select window/screen to share
3. Elegant sharing bar appears at top with pulsing indicator
4. Stop sharing via bar button or control bar button

**For Viewers:**
1. Video grid automatically replaced with shared screen
2. Large, centered view of shared content
3. Visual indicator shows who is sharing
4. Automatic return to video grid when sharing stops

### Key Benefits

1. **Seamless Experience**: Automatic switching between video and share views
2. **Reliable State**: Multiple event listeners ensure accurate share state
3. **WebCodecs Ready**: Video element approach works with modern web codecs
4. **Full Screen**: Share content uses full available space for optimal viewing
5. **Clean Transitions**: Smooth switching between video grid and share view
6. **Elegant Notifications**: Minimalist sharing bar provides clear status without blocking content

## Types

```typescript
// src/types/zoom.ts
export type ViewMode = "speaker" | "gallery";
export type ConnectionStatus =
  | "Disconnected"
  | "Connecting"
  | "Connected"
  | "Reconnecting"
  | "Failed";

export interface ChatMessage {
  id: string;
  sender: { name: string; userId: number };
  message: string;
  timestamp: number;
  isPrivate: boolean;
}
```

## Hooks

### useZoomClient.ts

```typescript
import ZoomVideo, { type VideoClient } from "@zoom/videosdk";
import { useCallback, useEffect, useRef, useState } from "react";

interface UseZoomClientOptions {
  onConnectionChange?: (state: string, reason?: string) => void;
}

export function useZoomClient(options: UseZoomClientOptions = {}) {
  const clientRef = useRef<typeof VideoClient | null>(null);
  const [client, setClient] = useState<typeof VideoClient | null>(null);
  const [isInitialized, setIsInitialized] = useState(false);
  const [isJoined, setIsJoined] = useState(false);

  // Store callback in ref to avoid recreating init on every render
  const onConnectionChangeRef = useRef(options.onConnectionChange);
  useEffect(() => {
    onConnectionChangeRef.current = options.onConnectionChange;
  });

  const getClient = useCallback(() => {
    if (!clientRef.current) {
      clientRef.current = ZoomVideo.createClient();
      setClient(clientRef.current);
    }
    return clientRef.current;
  }, []);

  const init = useCallback(async () => {
    const zoomClient = getClient();
    await zoomClient.init("en-US", "Global", {
      patchJsMedia: true,
      leaveOnPageUnload: true,
    });
    setIsInitialized(true);

    zoomClient.on("connection-change", (payload) => {
      onConnectionChangeRef.current?.(payload.state, payload.reason);
      if (payload.state === "Closed" || payload.state === "Fail") {
        setIsJoined(false);
      }
    });

    return zoomClient;
  }, [getClient]);

  const join = useCallback(
    async (
      sessionName: string,
      token: string,
      userName: string,
      password?: string
    ) => {
      const zoomClient = getClient();
      if (!isInitialized) {
        await init();
      }
      await zoomClient.join(sessionName, token, userName, password);
      setIsJoined(true);
      setClient(zoomClient);
      return zoomClient;
    },
    [getClient, init, isInitialized]
  );

  const leave = useCallback(async (endSession = false) => {
    const zoomClient = clientRef.current;
    if (zoomClient) {
      await zoomClient.leave(endSession);
      setIsJoined(false);
    }
  }, []);

  const destroy = useCallback(() => {
    if (clientRef.current) {
      ZoomVideo.destroyClient();
      clientRef.current = null;
      setClient(null);
      setIsInitialized(false);
      setIsJoined(false);
    }
  }, []);

  return {
    client,
    getClient,
    init,
    join,
    leave,
    destroy,
    isInitialized,
    isJoined,
  };
}
```

### useSessionStatus.ts

```typescript
import type { VideoClient } from "@zoom/videosdk";
import { useState } from "react";
import type { ConnectionStatus } from "@/types/zoom";

export function useSessionStatus(client: typeof VideoClient | null) {
  const [status, setStatus] = useState<ConnectionStatus>("Disconnected");

  return { status, setStatus };
}
```

### useMediaStream.ts

```typescript
import type { Stream, VideoClient } from "@zoom/videosdk";
import { useCallback, useEffect, useRef, useState } from "react";

export function useMediaStream(client: typeof VideoClient | null, isJoined: boolean) {
  const [stream, setStream] = useState<typeof Stream | null>(null);
  const [audioOn, setAudioOn] = useState(false);
  const [videoOn, setVideoOn] = useState(false);
  const [sharing, setSharing] = useState(false);
  const [activeShareUserId, setActiveShareUserId] = useState<number | null>(null);
  const [muted, setMuted] = useState(true);
  const [isAudioLoading, setIsAudioLoading] = useState(false);
  const [isVideoLoading, setIsVideoLoading] = useState(false);
  const shareVideoRef = useRef<HTMLVideoElement | null>(null);
  const streamRef = useRef<typeof Stream | null>(null);

  useEffect(() => {
    if (!client || !isJoined) return;

    const audioHandler = (payload: { action: string }) => {
      if (payload.action === "join") {
        setAudioOn(true);
        setMuted(true);
      } else if (payload.action === "leave") setAudioOn(false);
      else if (payload.action === "muted") setMuted(true);
      else if (payload.action === "unmuted") setMuted(false);
    };

    // CRITICAL: Sync video state with SDK events
    const videoActiveHandler = (payload: { state: string; userId: number }) => {
      const currentUser = client.getCurrentUserInfo();
      if (currentUser && payload.userId === currentUser.userId) {
        setVideoOn(payload.state === "Active");
      }
    };

    // CRITICAL: Track both own sharing state and who is actively sharing
    const shareHandler = (payload: { state: string; userId: number }) => {
      const currentUser = client.getCurrentUserInfo();

      // Update own sharing state
      if (currentUser && payload.userId === currentUser.userId) {
        setSharing(payload.state === "Active");
      }

      // Track who is actively sharing for viewing
      if (payload.state === "Active") {
        setActiveShareUserId(payload.userId);
      } else if (payload.state === "Inactive") {
        setActiveShareUserId(null);
      }
    };

    const peerShareHandler = (payload: { userId: number; action: string }) => {
      if (payload.action === "Start") {
        setActiveShareUserId(payload.userId);
      } else if (payload.action === "Stop") {
        setActiveShareUserId(null);
      }
    };

    const passivelyStopShareHandler = () => {
      setSharing(false);
      setActiveShareUserId(null);
    };

    client.on("current-audio-change", audioHandler);
    client.on("video-active-change", videoActiveHandler);

    // Screen sharing events
    client.on("active-share-change", shareHandler);
    client.on("peer-share-state-change", peerShareHandler);
    client.on("passively-stop-share", passivelyStopShareHandler);

    return () => {
      client.off("current-audio-change", audioHandler);
      client.off("video-active-change", videoActiveHandler);

      // Remove screen sharing event listeners
      client.off("active-share-change", shareHandler);
      client.off("peer-share-state-change", peerShareHandler);
      client.off("passively-stop-share", passivelyStopShareHandler);
    };
  }, [client, isJoined]);

  const initStream = useCallback(() => {
    if (client && isJoined) {
      const mediaStream = client.getMediaStream();
      setStream(mediaStream);
      streamRef.current = mediaStream;

      // CRITICAL: Create hidden video element for screen share (WebCodecs compatible)
      if (!shareVideoRef.current) {
        const video = document.createElement("video");
        video.style.display = "none";
        video.muted = true;
        video.autoplay = true;
        document.body.appendChild(video);
        shareVideoRef.current = video;
      }

      return mediaStream;
    }
    return null;
  }, [client, isJoined]);

  const toggleAudio = useCallback(async () => {
    const currentStream = streamRef.current || stream;
    if (!currentStream) throw new Error("Stream not initialized");

    try {
      setIsAudioLoading(true);

      if (!audioOn) {
        // Audio is not started yet, start it, unmuted by default
        await currentStream.startAudio();
        setAudioOn(true);
        setMuted(false);

        // Try to mute after starting
        try {
          await currentStream.muteAudio();
          setMuted(true);
        } catch (e) {
          console.log(e);
        }
      } else if (muted) {
        // Audio is on and muted, unmute it
        await currentStream.unmuteAudio();
        setMuted(false);
      } else {
        // Audio is on and unmuted, mute it
        await currentStream.muteAudio();
        setMuted(true);
      }
    } finally {
      setIsAudioLoading(false);
    }
  }, [stream, audioOn, muted]);

  const toggleVideo = useCallback(async () => {
    const currentStream = streamRef.current || stream;
    if (!currentStream) throw new Error("Stream not initialized");

    try {
      setIsVideoLoading(true);

      if (videoOn) {
        await currentStream.stopVideo();
        setVideoOn(false);
      } else {
        await currentStream.startVideo();
        setVideoOn(true);
      }
    } finally {
      setIsVideoLoading(false);
    }
  }, [stream, videoOn]);

  const toggleScreenShare = useCallback(async () => {
    const currentStream = streamRef.current || stream;
    if (!currentStream) throw new Error("Stream not initialized");

    if (!shareVideoRef.current) {
      throw new Error("Share video element not initialized");
    }

    if (sharing) {
      await currentStream.stopShareScreen();
      setSharing(false);
    } else {
      // CRITICAL: Use video element for WebCodecs compatibility
      await currentStream.startShareScreen(shareVideoRef.current);
      setSharing(true);
    }
  }, [stream, sharing]);

  // Cleanup video element on unmount
  useEffect(() => {
    return () => {
      if (shareVideoRef.current) {
        shareVideoRef.current.remove();
        shareVideoRef.current = null;
      }
    };
  }, []);

  return {
    stream,
    initStream,
    audioOn,
    muted,
    videoOn,
    sharing,
    activeShareUserId,
    isAudioLoading,
    isVideoLoading,
    toggleAudio,
    toggleVideo,
    toggleScreenShare,
  };
}
```

### useParticipants.ts

```typescript
import type { Participant, VideoClient } from '@zoom/videosdk'
import { useCallback, useEffect, useState } from 'react'

export function useParticipants(client: typeof VideoClient | null, isJoined = false) {
  const [participants, setParticipants] = useState<Participant[]>([])

  const updateList = useCallback(() => {
    if (client) {
      const users = client.getAllUser()
      setParticipants(users)
    }
  }, [client])

  useEffect(() => {
    if (!client || !isJoined) return

    updateList()

    client.on('user-added', updateList)
    client.on('user-updated', updateList)
    client.on('user-removed', updateList)
    client.on('current-audio-change', updateList)
    client.on('video-active-change', updateList)
    client.on('peer-video-state-change', updateList)
    client.on('connection-change', updateList)

    return () => {
      client.off('user-added', updateList)
      client.off('user-updated', updateList)
      client.off('user-removed', updateList)
      client.off('current-audio-change', updateList)
      client.off('video-active-change', updateList)
      client.off('peer-video-state-change', updateList)
      client.off('connection-change', updateList)
    }
  }, [client, isJoined, updateList])

  return { participants, refreshParticipants: updateList }
}
```

**Key Implementation Notes:**

- **Simple Approach**: All events trigger the same `updateList()` callback that calls `getAllUser()` to refresh the complete participant list
- **Event Coverage**: Listens to all participant-related events to ensure UI stays in sync with SDK state
- **Initial Fetch**: Calls `updateList()` immediately when session is joined to populate the initial participant list
- **Cleanup**: Properly removes all event listeners on unmount to prevent memory leaks
- **Export Pattern**: Returns both `participants` array and `refreshParticipants` callback for manual refresh if needed

### useChat.ts

```typescript
import type { VideoClient } from "@zoom/videosdk";
import { useCallback, useEffect, useState } from "react";
import type { ChatMessage } from "@/types/zoom";

export function useChat(client: typeof VideoClient | null, isJoined: boolean) {
  const [messages, setMessages] = useState<ChatMessage[]>([]);

  useEffect(() => {
    if (!client || !isJoined) return;

    const handler = (payload: {
      message: string;
      sender: { name: string; userId: number };
      timestamp: number;
      receiver: { userId: number };
    }) => {
      const isPrivate = payload.receiver.userId !== 0;
      setMessages((prev) => [
        ...prev,
        {
          id: crypto.randomUUID(),
          sender: { name: payload.sender.name, userId: payload.sender.userId },
          message: payload.message,
          timestamp: payload.timestamp,
          isPrivate,
        },
      ]);
    };

    client.on("chat-on-message", handler);

    return () => {
      client.off("chat-on-message", handler);
    };
  }, [client, isJoined]);

  const sendMessage = useCallback(
    async (text: string, toUserId?: number) => {
      if (!client || !isJoined) {
        throw new Error("Cannot send message: not connected");
      }

      const chatClient = client.getChatClient();

      if (toUserId) {
        await chatClient.send(text, toUserId);
      } else {
        await chatClient.sendToAll(text);
      }
      // Note: The message will appear in the chat via the 'chat-on-message' event
    },
    [client, isJoined]
  );

  const clearMessages = useCallback(() => {
    setMessages([]);
  }, []);

  return { messages, sendMessage, clearMessages };
}
```

**Key Implementation Notes:**

- **CRITICAL: Zoom SDK Echoes Sent Messages**: The SDK fires `chat-on-message` for the sender's own messages. Do NOT manually add sent messages to local state or they will appear twice.
- **isJoined Check**: Always verify `isJoined` before setting up event listeners and sending messages to prevent errors
- **Error Handling**: Wrap `sendMessage` in try/catch and log errors for debugging
- **Chat Privileges**: Check chat privileges with `getChatClient().getPrivilege()` to debug permission issues
- **Logging**: Include console logs during development to debug message flow issues
- **Private Messages**: Support optional `toUserId` parameter for direct messages

### useActiveSpeaker.ts

```typescript
import type { VideoClient } from "@zoom/videosdk";
import { useEffect, useState } from "react";

export function useActiveSpeaker(client: typeof VideoClient | null) {
  const [activeSpeakerId, setActiveSpeakerId] = useState<number | null>(null);

  useEffect(() => {
    if (!client) return;

    const handler = (payload: { activeSpeaker: Array<{ oderId: number }> }) => {
      if (payload.activeSpeaker.length > 0) {
        setActiveSpeakerId(payload.activeSpeaker[0].oderId);
      }
    };
    client.on("active-speaker", handler);
    return () => client.off("active-speaker", handler);
  }, [client]);

  return activeSpeakerId;
}
```

### hooks/index.ts

```typescript
/**
 * For optimal bundle size, import hooks directly from their files:
 * @example import { useChat } from '@/hooks/useChat'
 */
export { useActiveSpeaker } from "./useActiveSpeaker";
export { useChat } from "./useChat";
export { useMediaStream } from "./useMediaStream";
export { useParticipants } from "./useParticipants";
export { useSessionStatus } from "./useSessionStatus";
export { useZoomClient } from "./useZoomClient";
```

## Components

### VideoGrid.tsx

```tsx
import type { Stream } from "@zoom/videosdk";
import { MonitorPlay, VideoOff } from "lucide-react";
import { useEffect, useRef } from "react";
import { Badge } from "@/components/ui/badge";
import type { Participant, ViewMode } from "@/types/zoom";

interface VideoGridProps {
  participants: Participant[];
  viewMode: ViewMode;
  activeSpeakerId: number | null;
  activeShareUserId: number | null;
  stream: typeof Stream | null;
  currentUserId?: number;
  currentUserVideoOn?: boolean;
}

interface VideoTileProps {
  participant: Participant;
  stream: typeof Stream | null;
  isLarge?: boolean;
  isSpeaking?: boolean;
  currentUserId?: number;
  currentUserVideoOn?: boolean;
}

interface ShareViewProps {
  userId: number;
  stream: typeof Stream | null;
  userName: string;
}

// CRITICAL: ShareView component for rendering shared screen content
function ShareView({ userId, stream, userName }: ShareViewProps) {
  const containerRef = useRef<HTMLElement>(null);

  useEffect(() => {
    if (!stream || !containerRef.current || !userId) return;

    const container = containerRef.current;

    const attachShare = async () => {
      try {
        const shareElement = await stream.attachShareView(userId);

        if (shareElement) {
          container.innerHTML = "";
          const el = shareElement as unknown as HTMLElement;
          el.style.width = "100%";
          el.style.height = "100%";
          el.style.objectFit = "contain";
          container.appendChild(el);
        }
      } catch (error) {
        console.error("Error attaching share view:", error);
      }
    };

    attachShare();

    return () => {
      try {
        stream.detachShareView(userId);
        if (container) {
          container.innerHTML = "";
        }
      } catch (err) {
        console.error("Error detaching share view:", err);
      }
    };
  }, [stream, userId]);

  return (
    <div className="relative bg-black w-full h-full flex items-center justify-center">
      {/* CRITICAL: video-player-container is required by Zoom SDK */}
      <video-player-container
        ref={containerRef}
        className="w-full h-full"
        style={{ minHeight: "100%" }}
      />

      <div className="absolute top-4 left-4 bg-black/80 px-4 py-2 rounded-lg flex items-center gap-2 shadow-lg">
        <MonitorPlay className="h-5 w-5 text-green-400" />
        <span className="text-white text-sm font-medium">
          {userName} is sharing their screen
        </span>
      </div>
    </div>
  );
}

function VideoTile({
  participant,
  stream,
  isLarge = false,
  isSpeaking = false,
  currentUserId,
  currentUserVideoOn,
}: VideoTileProps) {
  const containerRef = useRef<HTMLElement>(null);

  // CRITICAL: For current user, use currentUserVideoOn prop
  const isCurrentUser = currentUserId === participant.userId;
  const isVideoOn = isCurrentUser
    ? currentUserVideoOn ?? participant.bVideoOn
    : participant.bVideoOn;

  useEffect(() => {
    if (!stream || !containerRef.current) return;

    const container = containerRef.current;

    if (!isVideoOn) {
      try {
        stream.detachVideo(participant.userId);
        container.innerHTML = "";
      } catch {
        // Ignore detach errors
      }
      return;
    }

    // CRITICAL: attachVideo returns element to append to video-player-container
    const attachVideo = async () => {
      try {
        const videoElement = await stream.attachVideo(
          participant.userId,
          3 // VideoQuality.Video_720P
        );

        if (videoElement) {
          container.innerHTML = "";
          const el = videoElement as unknown as HTMLElement;
          el.style.width = "100%";
          el.style.height = "100%";
          container.appendChild(el);
        }
      } catch (error) {
        console.error("Error attaching video:", error);
      }
    };

    attachVideo();

    return () => {
      try {
        stream.detachVideo(participant.userId);
      } catch {
        // Ignore detach errors
      }
    };
  }, [stream, isVideoOn, participant.userId, isCurrentUser]);

  return (
    <div
      className={`relative bg-muted rounded-lg overflow-hidden flex items-center justify-center ${
        isLarge ? "w-full h-full" : "aspect-video"
      } ${isSpeaking ? "ring-2 ring-green-500" : ""}`}
    >
      {/* CRITICAL: Must use video-player-container custom element */}
      <video-player-container
        ref={containerRef}
        className="w-full h-full"
        style={{ minHeight: "100px", display: isVideoOn ? "block" : "none" }}
      />

      {!isVideoOn && (
        <div className="flex flex-col items-center justify-center gap-2">
          <VideoOff className="h-5 w-5 text-muted-foreground" />
        </div>
      )}

      <div className="absolute bottom-2 left-2 right-2 flex items-center justify-between">
        <Badge variant="secondary" className="text-xs truncate max-w-[80%]">
          {participant.displayName}
          {participant.isHost && " (Host)"}
        </Badge>
      </div>
    </div>
  );
}

export function VideoGrid({
  participants,
  viewMode,
  activeSpeakerId,
  activeShareUserId,
  stream,
  currentUserId,
  currentUserVideoOn,
}: VideoGridProps) {
  // Find the participant who is sharing
  const sharingParticipant = activeShareUserId
    ? participants.find((p) => p.userId === activeShareUserId)
    : null;

  if (participants.length === 0) {
    return (
      <div className="flex items-center justify-center h-full">
        <p className="text-muted-foreground">Waiting for participants...</p>
      </div>
    );
  }

  // CRITICAL: If someone is sharing, show only the share view (no video grid)
  if (sharingParticipant && stream) {
    return (
      <div className="h-full">
        <ShareView
          userId={sharingParticipant.userId}
          stream={stream}
          userName={sharingParticipant.displayName}
        />
      </div>
    );
  }

  if (viewMode === "speaker") {
    const speaker =
      participants.find((p) => p.userId === activeSpeakerId) || participants[0];
    const others = participants.filter((p) => p.userId !== speaker.userId);

    return (
      <div className="relative h-full p-4">
        <VideoTile
          participant={speaker}
          stream={stream}
          isLarge
          isSpeaking={speaker.userId === activeSpeakerId}
          currentUserId={currentUserId}
          currentUserVideoOn={currentUserVideoOn}
        />
        {others.length > 0 && (
          <div className="absolute bottom-6 left-6 flex gap-2">
            {others.slice(0, 4).map((p) => (
              <div key={p.userId} className="w-32">
                <VideoTile
                  participant={p}
                  stream={stream}
                  isSpeaking={p.userId === activeSpeakerId}
                  currentUserId={currentUserId}
                  currentUserVideoOn={currentUserVideoOn}
                />
              </div>
            ))}
          </div>
        )}
      </div>
    );
  }

  const getGridCols = (count: number) => {
    if (count === 1) return "grid-cols-1";
    if (count <= 4) return "grid-cols-2";
    if (count <= 9) return "grid-cols-3";
    return "grid-cols-4";
  };

  return (
    <div
      className={`grid ${getGridCols(participants.length)} gap-2 h-full p-4`}
    >
      {participants.map((p) => (
        <VideoTile
          key={p.userId}
          participant={p}
          stream={stream}
          isSpeaking={p.userId === activeSpeakerId}
          currentUserId={currentUserId}
          currentUserVideoOn={currentUserVideoOn}
        />
      ))}
    </div>
  );
}
```

### SessionStatus.tsx

```tsx
import { Badge } from "@/components/ui/badge";
import type { ConnectionStatus } from "@/types/zoom";

interface SessionStatusProps {
  status: ConnectionStatus;
}

export function SessionStatus({ status }: SessionStatusProps) {
  const variants: Record<ConnectionStatus, string> = {
    Connected: "bg-green-500 text-white",
    Connecting: "bg-yellow-500 animate-pulse",
    Reconnecting: "bg-yellow-500 animate-pulse",
    Disconnected: "bg-gray-500",
    Failed: "bg-red-500 text-white",
  };

  return <Badge className={variants[status]}>{status}</Badge>;
}
```

### ControlBar.tsx

```tsx
import {
  Headphones,
  Info,
  Loader2,
  LogOut,
  MessageSquare,
  Mic,
  MicOff,
  Monitor,
  Users,
  Video,
  VideoOff,
} from "lucide-react";
import { Button } from "@/components/ui/button";

interface ControlBarProps {
  audioOn: boolean;
  muted: boolean;
  videoOn: boolean;
  sharing: boolean;
  isAudioLoading: boolean;
  isVideoLoading: boolean;
  onToggleAudio: () => void;
  onToggleVideo: () => void;
  onToggleShare: () => void;
  onLeave: () => void;
  onToggleChat: () => void;
  onToggleParticipants: () => void;
  onToggleInfo: () => void;
  showChat: boolean;
  showParticipants: boolean;
}

export function ControlBar({
  audioOn,
  muted,
  videoOn,
  sharing,
  isAudioLoading,
  isVideoLoading,
  onToggleAudio,
  onToggleVideo,
  onToggleShare,
  onLeave,
  onToggleChat,
  onToggleParticipants,
  onToggleInfo,
  showChat,
  showParticipants,
}: ControlBarProps) {
  return (
    <div className="flex items-center justify-center gap-2 p-4 border-t bg-background">
      {/* Audio Button with Loading State */}
      <Button
        variant={audioOn && !muted ? "default" : "secondary"}
        size="icon"
        onClick={onToggleAudio}
        disabled={isAudioLoading}
      >
        {isAudioLoading ? (
          <Loader2 className="h-5 w-5 animate-spin" />
        ) : !audioOn ? (
          <Headphones className="h-5 w-5" />
        ) : muted ? (
          <MicOff className="h-5 w-5" />
        ) : (
          <Mic className="h-5 w-5" />
        )}
      </Button>

      {/* Video Button with Loading State */}
      <Button
        variant={videoOn ? "default" : "secondary"}
        size="icon"
        onClick={onToggleVideo}
        disabled={isVideoLoading}
      >
        {isVideoLoading ? (
          <Loader2 className="h-5 w-5 animate-spin" />
        ) : videoOn ? (
          <Video className="h-5 w-5" />
        ) : (
          <VideoOff className="h-5 w-5" />
        )}
      </Button>

      <Button
        variant={sharing ? "default" : "secondary"}
        size="icon"
        onClick={onToggleShare}
      >
        <Monitor className="h-5 w-5" />
      </Button>

      <div className="w-px h-8 bg-border mx-2" />

      <Button
        variant={showParticipants ? "default" : "ghost"}
        size="icon"
        onClick={onToggleParticipants}
      >
        <Users className="h-5 w-5" />
      </Button>

      <Button
        variant={showChat ? "default" : "ghost"}
        size="icon"
        onClick={onToggleChat}
      >
        <MessageSquare className="h-5 w-5" />
      </Button>

      <Button variant="ghost" size="icon" onClick={onToggleInfo}>
        <Info className="h-5 w-5" />
      </Button>

      <div className="w-px h-8 bg-border mx-2" />

      <Button variant="destructive" onClick={onLeave}>
        <LogOut className="h-4 w-4 mr-2" />
        Leave
      </Button>
    </div>
  );
}
```

### ParticipantList.tsx

```tsx
import type { Participant } from "@zoom/videosdk";
import { Mic, MicOff, Video } from "lucide-react";
import { Avatar, AvatarFallback } from "@/components/ui/avatar";
import { Badge } from "@/components/ui/badge";
import { ScrollArea } from "@/components/ui/scroll-area";

interface ParticipantListProps {
  participants: Participant[];
}

export function ParticipantList({ participants }: ParticipantListProps) {
  return (
    <div className="flex flex-col h-full">
      <div className="p-3 border-b">
        <h3 className="font-semibold">Participants ({participants.length})</h3>
      </div>
      <ScrollArea className="flex-1">
        <div className="p-2 space-y-1">
          {participants.map((user) => (
            <div
              key={user.userId}
              className="flex items-center gap-2 p-2 rounded-md hover:bg-muted"
            >
              <Avatar className="h-8 w-8">
                <AvatarFallback>{user.displayName[0]}</AvatarFallback>
              </Avatar>
              <span className="flex-1 text-sm truncate">
                {user.displayName}
              </span>
              {user.isHost && (
                <Badge variant="secondary" className="text-xs">
                  Host
                </Badge>
              )}
              {user.bVideoOn && (
                <Video className="h-4 w-4 text-muted-foreground" />
              )}
              {user.muted ? (
                <MicOff className="h-4 w-4 text-muted-foreground" />
              ) : (
                <Mic className="h-4 w-4 text-muted-foreground" />
              )}
            </div>
          ))}
        </div>
      </ScrollArea>
    </div>
  );
}
```

### ChatPanel.tsx

```tsx
import { Send } from "lucide-react";
import { type FormEvent, useState } from "react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { ScrollArea } from "@/components/ui/scroll-area";
import type { ChatMessage } from "@/types/zoom";

interface ChatPanelProps {
  messages: ChatMessage[];
  onSend: (text: string) => void;
}

export function ChatPanel({ messages, onSend }: ChatPanelProps) {
  const [input, setInput] = useState("");

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    if (input.trim()) {
      onSend(input.trim());
      setInput("");
    }
  };

  return (
    <div className="flex flex-col h-full">
      <div className="p-3 border-b">
        <h3 className="font-semibold">Chat</h3>
      </div>
      <ScrollArea className="flex-1 p-3">
        <div className="space-y-3">
          {messages.map((msg) => (
            <div key={msg.id} className="text-sm">
              <span className="font-medium">{msg.sender.name}: </span>
              <span>{msg.message}</span>
            </div>
          ))}
        </div>
      </ScrollArea>
      <form onSubmit={handleSubmit} className="p-3 border-t flex gap-2">
        <Input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Type a message..."
          className="flex-1"
        />
        <Button type="submit" size="icon">
          <Send className="h-4 w-4" />
        </Button>
      </form>
    </div>
  );
}
```

### JoinForm.tsx

```tsx
import { Loader2, Video } from "lucide-react";
import { type FormEvent, useCallback, useEffect, useRef, useState } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";

interface JoinFormProps {
  onJoin: (sessionName: string, userName: string, token: string) => void;
  isLoading?: boolean;
}

export function JoinForm({ onJoin, isLoading }: JoinFormProps) {
  const [sessionName, setSessionName] = useState("example-session");
  const [userName, setUserName] = useState("example-user");
  const [token, setToken] = useState("");
  const [isFetchingToken, setIsFetchingToken] = useState(false);
  const [tokenError, setTokenError] = useState<string | null>(null);
  const hasAutoJoinedRef = useRef(false);

  const fetchToken = useCallback(async (session: string) => {
    if (!session) return;

    setIsFetchingToken(true);
    setTokenError(null);

    try {
      const response = await fetch("http://localhost:4000", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ sessionName: session, role: 1 }),
      });

      if (!response.ok) throw new Error("Failed to get token");

      const data = await response.json();
      setToken(data.signature);
    } catch (error) {
      setTokenError("Failed to get token. Make sure your server is running.");
    } finally {
      setIsFetchingToken(false);
    }
  }, []);

  // CRITICAL: Check URL parameters and auto-join if present
  useEffect(() => {
    // Prevent multiple auto-join attempts
    if (hasAutoJoinedRef.current) {
      return;
    }

    const urlParams = new URLSearchParams(window.location.search);
    const urlSessionName = urlParams.get("sessionName");
    const urlUserName = urlParams.get("userName");
    const urlToken = urlParams.get("token");

    // Set form fields from URL parameters if present
    if (urlSessionName) {
      setSessionName(urlSessionName);
    }
    if (urlUserName) {
      setUserName(urlUserName);
    }
    if (urlToken) {
      setToken(urlToken);
    }

    // If all required parameters are in URL, auto-join with delay to ensure client is ready
    if (urlSessionName && urlUserName && urlToken) {
      console.log("Scheduling auto-join with URL parameters:", {
        urlSessionName,
        urlUserName,
        urlToken,
      });
      hasAutoJoinedRef.current = true;

      // Add a longer delay to ensure the Zoom client is fully initialized
      // This gives the client time to create and be ready for joining
      setTimeout(() => {
        console.log("Executing auto-join now...");
        onJoin(urlSessionName, urlUserName, urlToken);
      }, 500);
      return;
    }

    // Otherwise, auto fetch token on mount if sessionName exists (only once)
    if (sessionName && !hasAutoJoinedRef.current) {
      hasAutoJoinedRef.current = true;
      fetchToken(sessionName);
    }
  }, [onJoin, sessionName, fetchToken]);

  const handleGetToken = () => {
    fetchToken(sessionName);
  };

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    if (sessionName && userName && token) {
      onJoin(sessionName, userName, token);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-background p-4">
      <Card className="w-full max-w-md">
        <CardHeader className="text-center">
          <div className="mx-auto mb-4 h-12 w-12 rounded-full bg-primary/10 flex items-center justify-center">
            <Video className="h-6 w-6 text-primary" />
          </div>
          <CardTitle className="text-2xl">Join Video Session</CardTitle>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit} className="space-y-4">
            <div className="space-y-2">
              <label className="text-sm font-medium">Session Name</label>
              <Input
                value={sessionName}
                onChange={(e) => setSessionName(e.target.value)}
                placeholder="Enter session name"
                required
              />
            </div>
            <div className="space-y-2">
              <label className="text-sm font-medium">Your Name</label>
              <Input
                value={userName}
                onChange={(e) => setUserName(e.target.value)}
                placeholder="Enter your name"
                required
              />
            </div>
            <div className="space-y-2">
              <label className="text-sm font-medium">JWT Token</label>
              <div className="flex gap-2">
                <Input
                  value={token}
                  onChange={(e) => setToken(e.target.value)}
                  placeholder={isFetchingToken ? "Fetching..." : "Token auto-fetched"}
                  required
                  className="flex-1"
                />
                <Button
                  type="button"
                  variant="secondary"
                  onClick={handleGetToken}
                  disabled={isFetchingToken || !sessionName}
                >
                  {isFetchingToken ? (
                    <Loader2 className="h-4 w-4 animate-spin" />
                  ) : (
                    "Refresh"
                  )}
                </Button>
              </div>
              {tokenError && (
                <p className="text-xs text-destructive">{tokenError}</p>
              )}
              {token && !tokenError && (
                <p className="text-xs text-green-600">Token ready</p>
              )}
            </div>
            <Button
              type="submit"
              className="w-full"
              disabled={isLoading || !token}
            >
              {isLoading ? "Joining..." : "Join Session"}
            </Button>
          </form>
        </CardContent>
      </Card>
    </div>
  );
}
```

### SessionInfo.tsx

```tsx
import type { VideoClient } from "@zoom/videosdk";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";

interface SessionInfoProps {
  client: typeof VideoClient | null;
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

export function SessionInfo({ client, open, onOpenChange }: SessionInfoProps) {
  if (!client) return null;

  const sessionInfo = client.getSessionInfo();
  const currentUser = client.getCurrentUserInfo();

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Session Information</DialogTitle>
        </DialogHeader>
        <div className="grid gap-2 text-sm">
          <div>
            <span className="font-medium">Session:</span> {sessionInfo.topic}
          </div>
          <div>
            <span className="font-medium">Session ID:</span>{" "}
            {sessionInfo.sessionId}
          </div>
          <div>
            <span className="font-medium">Your Name:</span>{" "}
            {currentUser?.displayName}
          </div>
          <div>
            <span className="font-medium">Your Role:</span>{" "}
            {currentUser?.isHost
              ? "Host"
              : currentUser?.isManager
              ? "Manager"
              : "Participant"}
          </div>
          <div>
            <span className="font-medium">Participants:</span>{" "}
            {client.getAllUser().length}
          </div>
        </div>
      </DialogContent>
    </Dialog>
  );
}
```

### SharingBar.tsx

```tsx
import { MonitorOff, MonitorPlay } from 'lucide-react'
import { Button } from '@/components/ui/button'

interface SharingBarProps {
  onStopSharing: () => void
}

export function SharingBar({ onStopSharing }: SharingBarProps) {
  return (
    <div className="fixed top-4 left-1/2 -translate-x-1/2 z-50 animate-in fade-in slide-in-from-top-2 duration-300">
      <div className="bg-gradient-to-r from-green-600 to-green-500 text-white rounded-full shadow-xl flex items-center gap-3 px-5 py-2.5 backdrop-blur-sm">
        {/* Status indicator with animated icon */}
        <div className="flex items-center gap-2.5">
          <div className="relative">
            <MonitorPlay className="h-4 w-4" />
            <span className="absolute -top-1 -right-1 h-2 w-2 bg-white rounded-full animate-pulse" />
          </div>
          <span className="font-medium text-sm">You're sharing your screen</span>
        </div>

        {/* Stop button */}
        <Button
          variant="secondary"
          size="sm"
          onClick={onStopSharing}
          className="h-7 text-xs font-medium hover:bg-white/90 transition-all"
        >
          <MonitorOff className="h-3.5 w-3.5 mr-1.5" />
          Stop
        </Button>
      </div>
    </div>
  )
}
```

### SessionPage.tsx (Main Page)

```tsx
import { Grid2X2, LayoutGrid } from "lucide-react";
import { useCallback, useEffect, useState } from "react";
import { Button } from "@/components/ui/button";
import {
  useActiveSpeaker,
  useChat,
  useMediaStream,
  useParticipants,
  useSessionStatus,
  useZoomClient,
} from "@/hooks";
import type { ViewMode } from "@/types/zoom";
import { ChatPanel } from "./ChatPanel";
import { ControlBar } from "./ControlBar";
import { JoinForm } from "./JoinForm";
import { ParticipantList } from "./ParticipantList";
import { SessionInfo } from "./SessionInfo";
import { SessionStatus } from "./SessionStatus";
import { SharingBar } from "./SharingBar";
import { VideoGrid } from "./VideoGrid";

export function SessionPage() {
  const { client, getClient, join, leave, destroy, isJoined } = useZoomClient();
  const { status, setStatus } = useSessionStatus(client);
  const {
    stream,
    initStream,
    audioOn,
    muted,
    videoOn,
    sharing,
    activeShareUserId,
    isAudioLoading,
    isVideoLoading,
    toggleAudio,
    toggleVideo,
    toggleScreenShare,
  } = useMediaStream(client, isJoined);
  const { participants } = useParticipants(client, isJoined);
  const activeSpeakerId = useActiveSpeaker(client);
  const { messages, sendMessage, clearMessages } = useChat(client, isJoined);

  const [viewMode, setViewMode] = useState<ViewMode>("gallery");
  const [showChat, setShowChat] = useState(false);
  const [showParticipants, setShowParticipants] = useState(false);
  const [showInfo, setShowInfo] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(false);

  const currentUser = client?.getCurrentUserInfo();

  useEffect(() => {
    if (isJoined && client && !stream) {
      initStream();
    }
  }, [isJoined, client, stream, initStream]);

  const handleJoin = async (
    sessionName: string,
    userName: string,
    token: string
  ) => {
    try {
      setIsLoading(true);
      setError(null);
      setStatus("Connecting");

      console.log("Attempting to join session:", {
        sessionName,
        userName,
        tokenLength: token.length,
      });

      await join(sessionName, token, userName);
      setStatus("Connected");

      // CRITICAL: Add parameters to URL after successful join
      const url = new URL(window.location.href);
      url.searchParams.set("sessionName", sessionName);
      url.searchParams.set("userName", userName);
      url.searchParams.set("token", token);
      window.history.pushState({}, "", url.toString());
    } catch (err) {
      console.error("Join error details:", err);
      console.error("Error type:", typeof err);
      console.error("Error keys:", err ? Object.keys(err) : "null");

      let errorMessage = "Failed to join session";
      if (err instanceof Error) {
        errorMessage = err.message || err.toString();
      } else if (typeof err === "string") {
        errorMessage = err;
      } else if (err && typeof err === "object") {
        errorMessage = JSON.stringify(err);
      }

      setError(errorMessage);
      setStatus("Failed");
    } finally {
      setIsLoading(false);
    }
  };

  const handleLeave = async () => {
    try {
      clearMessages();
      await leave();

      // CRITICAL: Remove parameters from URL after leaving
      const url = new URL(window.location.href);
      url.searchParams.delete("sessionName");
      url.searchParams.delete("userName");
      url.searchParams.delete("token");
      window.history.pushState({}, "", url.toString());
    } catch (err) {
      console.error("Leave error:", err);
    }
  };

  const handleToggleAudio = useCallback(async () => {
    try {
      setError(null);
      await toggleAudio();
    } catch (err) {
      setError(err instanceof Error ? err.message : "Audio error");
      setTimeout(() => setError(null), 3000);
    }
  }, [toggleAudio]);

  const handleToggleVideo = useCallback(async () => {
    try {
      setError(null);
      await toggleVideo();
    } catch (err) {
      setError(err instanceof Error ? err.message : "Video error");
      setTimeout(() => setError(null), 3000);
    }
  }, [toggleVideo]);

  const handleToggleScreenShare = useCallback(async () => {
    try {
      setError(null);
      await toggleScreenShare();
    } catch (err) {
      setError(err instanceof Error ? err.message : "Screen share error");
      setTimeout(() => setError(null), 3000);
    }
  }, [toggleScreenShare]);

  // CRITICAL: Empty deps to prevent re-renders from causing client destruction
  useEffect(() => {
    getClient();
    return () => {
      // Only destroy on actual unmount, not on re-renders
      destroy();
    };
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []); // Empty deps - only run on mount/unmount

  if (!isJoined) {
    return (
      <div>
        <JoinForm onJoin={handleJoin} isLoading={isLoading} />
        {error && (
          <div className="fixed bottom-4 left-1/2 -translate-x-1/2 bg-destructive text-white px-4 py-2 rounded-md">
            {error}
          </div>
        )}
      </div>
    );
  }

  return (
    <div className="flex flex-col h-screen bg-background">
      <header className="flex items-center justify-between px-4 py-2 border-b">
        <SessionStatus status={status} />
        <div className="flex gap-1">
          <Button
            variant={viewMode === "speaker" ? "default" : "ghost"}
            size="sm"
            onClick={() => setViewMode("speaker")}
          >
            <LayoutGrid className="h-4 w-4 mr-1" />
            Speaker
          </Button>
          <Button
            variant={viewMode === "gallery" ? "default" : "ghost"}
            size="sm"
            onClick={() => setViewMode("gallery")}
          >
            <Grid2X2 className="h-4 w-4 mr-1" />
            Gallery
          </Button>
        </div>
      </header>

      <div className="flex flex-1 overflow-hidden">
        <main className="flex-1 overflow-hidden">
          {/* CRITICAL: Pass videoOn, currentUserId, and activeShareUserId to VideoGrid */}
          <VideoGrid
            participants={participants}
            viewMode={viewMode}
            activeSpeakerId={activeSpeakerId}
            activeShareUserId={activeShareUserId}
            stream={stream}
            currentUserId={currentUser?.userId}
            currentUserVideoOn={videoOn}
          />
        </main>

        {showParticipants && (
          <aside className="w-64 border-l overflow-hidden">
            <ParticipantList participants={participants} />
          </aside>
        )}

        {showChat && (
          <aside className="w-80 border-l overflow-hidden">
            <ChatPanel messages={messages} onSend={sendMessage} />
          </aside>
        )}
      </div>

      <ControlBar
        audioOn={audioOn}
        muted={muted}
        videoOn={videoOn}
        sharing={sharing}
        isAudioLoading={isAudioLoading}
        isVideoLoading={isVideoLoading}
        onToggleAudio={handleToggleAudio}
        onToggleVideo={handleToggleVideo}
        onToggleShare={handleToggleScreenShare}
        onLeave={handleLeave}
        onToggleChat={() => setShowChat(!showChat)}
        onToggleParticipants={() => setShowParticipants(!showParticipants)}
        onToggleInfo={() => setShowInfo(true)}
        showChat={showChat}
        showParticipants={showParticipants}
      />

      <SessionInfo client={client} open={showInfo} onOpenChange={setShowInfo} />

      {/* Show sharing bar when current user is sharing */}
      {sharing && (
        <SharingBar onStopSharing={handleToggleScreenShare} />
      )}

      {error && (
        <div className="fixed bottom-20 left-1/2 -translate-x-1/2 bg-destructive text-white px-4 py-2 rounded-md z-50">
          {error}
        </div>
      )}
    </div>
  );
}
```

## App.tsx

```tsx
import { SessionPage } from "@/components/video-session/SessionPage";

function App() {
  return <SessionPage />;
}

export default App;
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Auto-Join from URL Fails with OPERATION_CANCELLED Error

**Symptom**: When using URL parameters to auto-join, you get `{"type":"OPERATION_CANCELLED","reason":"LEAVING_MEETING","errorCode":3}`

**Cause**: Zoom client not fully initialized when auto-join attempted

**Solution**:
- Use 500ms delay before auto-join (not 100ms)
- Ensure client initialization uses empty dependency array to prevent re-renders
- Check that `hasAutoJoinedRef` prevents multiple join attempts

```typescript
// In JoinForm.tsx
useEffect(() => {
  if (hasAutoJoinedRef.current) return;
  // ... URL parameter parsing
  if (urlSessionName && urlUserName && urlToken) {
    hasAutoJoinedRef.current = true;
    setTimeout(() => {
      onJoin(urlSessionName, urlUserName, urlToken);
    }, 500); // 500ms delay is critical
  }
}, [onJoin]);
```

#### 2. Participant List Not Updating When Users Join/Leave

**Symptom**: New participants don't appear in the list, or left participants still show

**Cause**: Missing event listeners or not calling `getAllUser()` to refresh the list

**Solution**:
- Listen to all relevant events: `user-added`, `user-updated`, `user-removed`
- Call `getAllUser()` in response to these events to get the updated participant list
- Ensure event listeners are properly registered after joining the session

```typescript
// CORRECT - Listen to all user events and refresh list
const updateList = useCallback(() => {
  if (client) {
    const users = client.getAllUser();
    setParticipants(users);
  }
}, [client]);

client.on('user-added', updateList);
client.on('user-updated', updateList);
client.on('user-removed', updateList);
```

#### 2b. Current User's Audio/Video Status Not Updating in Participant List

**Symptom**: When you change your own audio/video, the participant list icons don't update immediately

**Cause**: Not listening to media state change events that affect the current user

**Solution**:
- Listen to `current-audio-change` event for the current user's audio state changes
- Listen to `video-active-change` and `peer-video-state-change` for video state changes
- Call `getAllUser()` in response to refresh the participant list with updated states

```typescript
// CRITICAL: Must listen to media events for current user's state
const updateList = useCallback(() => {
  if (client) {
    const users = client.getAllUser();
    setParticipants(users);
  }
}, [client]);

client.on('current-audio-change', updateList);  // Current user's audio
client.on('video-active-change', updateList);    // All users' video
client.on('peer-video-state-change', updateList); // Peer video changes
```

#### 3. Audio Button Stuck in Wrong State

**Symptom**: Audio is started but button shows "Join Computer Audio"

**Cause**: State updates happening before SDK operations complete

**Solution**:
- Use async/await for sequential state updates
- Update state only after SDK operations complete successfully

```typescript
// CORRECT - Sequential async/await updates
await currentStream.startAudio();
setAudioOn(true);
await currentStream.muteAudio();
setMuted(true);

// INCORRECT - State updated before awaiting
currentStream.startAudio(); // Missing await!
setAudioOn(true); // May update before audio actually starts
```

#### 4. Audio Starts Unmuted Instead of Muted

**Symptom**: Users join with microphone unmuted even though code tries to mute

**Cause**: Zoom SDK's `startAudio()` joins audio **unmuted by default**


**Solution**:
- Explicitly call `muteAudio()` after `startAudio()` completes
- Wrap in try/catch for graceful error handling

```typescript
await currentStream.startAudio();
setAudioOn(true);

try {
  await currentStream.muteAudio();
  setMuted(true);
} catch {
  setMuted(false); // Update state to reflect actual status
}
```

#### 5. Video Not Detaching When User Stops Video

**Symptom**: Video element remains visible after user stops their video

**Cause**: Not listening to `video-active-change` and `peer-video-state-change` events

**Solution**:
- Listen to both video state events
- Call `getAllUser()` to refresh participant list with updated `bVideoOn` status

```typescript
client.on('video-active-change', () => updateList());
client.on('peer-video-state-change', () => updateList());
```

#### 6. SharedArrayBuffer Errors in Console

**Symptom**: `SharedArrayBuffer is not defined` or similar COOP/COEP errors

**Cause**: Missing COOP/COEP headers in Vite configuration

**Solution**: Add headers to `vite.config.ts`:

```typescript
server: {
  headers: {
    "Cross-Origin-Opener-Policy": "same-origin",
    "Cross-Origin-Embedder-Policy": "require-corp",
  },
},
```

#### 7. Screen Share Not Displaying

**Symptom**: Screen share button works but shared screen doesn't display for viewers

**Cause**: Missing screen share event listeners or incorrect state tracking

**Solution**:
- Listen to all screen sharing events: `active-share-change`, `peer-share-state-change`, `passively-stop-share`
- Track `activeShareUserId` state separately from own `sharing` state
- Pass `activeShareUserId` to VideoGrid component
- Ensure ShareView component uses `attachShareView()` method

```typescript
// Track both own sharing and active sharer
const shareHandler = (payload: { state: string; userId: number }) => {
  const currentUser = client.getCurrentUserInfo();

  // Update own sharing state
  if (currentUser && payload.userId === currentUser.userId) {
    setSharing(payload.state === "Active");
  }

  // Track who is actively sharing
  if (payload.state === "Active") {
    setActiveShareUserId(payload.userId);
  } else if (payload.state === "Inactive") {
    setActiveShareUserId(null);
  }
};

client.on('active-share-change', shareHandler);
client.on('peer-share-state-change', peerShareHandler);
client.on('passively-stop-share', passivelyStopShareHandler);
```

#### 8. Canvas vs Video Element for Screen Sharing

**Symptom**: Screen sharing fails with WebCodecs or modern browser features

**Cause**: Using `<canvas>` instead of `<video>` element for screen sharing

**Solution**:
- Use `<video>` element instead of `<canvas>` for WebCodecs compatibility
- Set proper attributes: `muted`, `autoplay`, `display: none`
- Create element in `initStream()` and clean up on unmount

```typescript
// CORRECT - Video element approach
const video = document.createElement("video");
video.style.display = "none";
video.muted = true;
video.autoplay = true;
document.body.appendChild(video);
await stream.startShareScreen(video);

// INCORRECT - Canvas approach (legacy)
const canvas = document.createElement("canvas");
await stream.startShareScreen(canvas);
```

## Best Practices

### State Management

1. **Use Async/Await**: Always await SDK operations before updating state
2. **Single Source of Truth**: Let React state drive UI, don't query SDK state repeatedly
3. **Loading States**: Show loading indicators during async operations
4. **Error Handling**: Use try/finally to ensure loading states are cleared

### Client Lifecycle

1. **Empty Dependency Arrays**: Use `[]` for client initialization to prevent re-renders
2. **Proper Cleanup**: Always destroy client in unmount cleanup
3. **Auto-Join Delays**: Use 500ms delay for auto-join to ensure client ready

### Event Handling

1. **Use Event Payloads**: Participant events provide complete updated lists
2. **Consolidate Handlers**: Reuse the same handler function for events with identical logic (e.g., all user events)
3. **Named Handlers**: Use named functions for event handlers to enable proper cleanup
4. **Comprehensive Listening**: Listen to all relevant events (video, audio, user changes)

### Audio/Video Controls

1. **Explicit Muting**: Always call `muteAudio()` after `startAudio()` for privacy
2. **Promise Chaining**: Chain operations to ensure correct execution order
3. **Icon States**: Use proper icon components (Headphones, Mic, MicOff, Video, VideoOff)
4. **Disabled During Loading**: Disable buttons while operations in progress

### URL Parameters

1. **Validation**: Check all required parameters present before auto-join
2. **Prevent Multiple Attempts**: Use `useRef` to prevent multiple auto-joins
3. **Update URL**: Add parameters after successful join for sharing
4. **Clean Removal**: Remove parameters when leaving session

### Screen Sharing

1. **Video Element**: Use `<video>` element instead of `<canvas>` for WebCodecs compatibility
2. **Event Listeners**: Listen to all share events (`active-share-change`, `peer-share-state-change`, `passively-stop-share`)
3. **State Tracking**: Track both own sharing state and `activeShareUserId` separately
4. **ShareView Component**: Use dedicated component with `attachShareView()` for rendering
5. **Full-Screen Display**: Show share view exclusively when someone is sharing

### Performance

1. **Video Element Reuse**: Create screen share video element once and reuse (WebCodecs compatible)
2. **Video Detachment**: Always detach video when stopping to free resources
3. **Event Cleanup**: Remove all event listeners in cleanup functions
4. **Minimal Re-renders**: Use `useCallback` and `useMemo` appropriately

## Quick Reference

### Essential Patterns

**Async/Await State Updates:**
```typescript
await stream.startAudio();
setAudioOn(true);

try {
  await stream.muteAudio();
  setMuted(true);
} catch {
  setMuted(false);
}
```

**Participant List Updates:**
```typescript
// Single updateList callback for all events
const updateList = useCallback(() => {
  if (client) {
    const users = client.getAllUser();
    setParticipants(users);
  }
}, [client]);

// Register all relevant event listeners
client.on('user-added', updateList);
client.on('user-updated', updateList);
client.on('user-removed', updateList);

// CRITICAL: Also listen to media changes for current user
client.on('current-audio-change', updateList);  // Current user's audio
client.on('video-active-change', updateList);    // All users' video
client.on('peer-video-state-change', updateList); // Peer video changes
```

**Auto-Join with URL Parameters:**
```typescript
useEffect(() => {
  const params = new URLSearchParams(window.location.search);
  const sessionName = params.get('sessionName');
  const userName = params.get('userName');
  const token = params.get('token');

  if (sessionName && userName && token) {
    setTimeout(() => onJoin(sessionName, userName, token), 500);
  }
}, [onJoin]);
```

**Loading States:**
```typescript
const [isLoading, setIsLoading] = useState(false);

const handleAction = async () => {
  try {
    setIsLoading(true);
    await stream.startAudio();
  } finally {
    setIsLoading(false);
  }
};
```

### Required Icons (lucide-react)

```typescript
import {
  Headphones,    // Audio not joined
  Mic,           // Audio unmuted
  MicOff,        // Audio muted
  Video,         // Video on
  VideoOff,      // Video off
  Monitor,       // Screen share button
  MonitorPlay,   // Screen share indicator in ShareView and SharingBar
  MonitorOff,    // Stop sharing button in SharingBar
  Users,         // Participants
  MessageSquare, // Chat
  Info,          // Info dialog
  LogOut,        // Leave session
  Loader2,       // Loading spinner
} from 'lucide-react';
```

### Critical Timing Values

- **Auto-join delay**: 500ms (not 100ms) - ensures Zoom client is fully initialized
- **Initial participant fetch**: Immediate on join - no delay needed
- **Client initialization**: Empty dependency array `[]` - prevents re-initialization

### Event Listeners Checklist

✅ Audio Events:
- `current-audio-change` - **CRITICAL for participant list**: Current user's audio state, triggers participant list refresh
- `active-speaker` - Speaking indicator
- `speaking-while-muted` - User feedback

✅ Video Events:
- `video-active-change` - **CRITICAL for participant list**: All users' video state, triggers participant list refresh
- `peer-video-state-change` - Other users' video changes, triggers participant list refresh

✅ Participant Events:
- `user-added` - New participant joined, triggers participant list refresh
- `user-updated` - Participant state changed, triggers participant list refresh
- `user-removed` - Participant left, triggers participant list refresh

✅ Connection Events:
- `connection-change` - Connection status, triggers participant list refresh

✅ Share Events:
- `active-share-change` - **CRITICAL**: Screen share state, tracks both own sharing and active sharer
- `peer-share-state-change` - **CRITICAL**: Peer screen sharing state changes, tracks who is sharing
- `passively-stop-share` - **CRITICAL**: Forced screen share stop, resets sharing state
- `share-content-change` - Screen share content updates (optional)
- `share-content-dimension-change` - Screen share dimension changes (optional)

✅ Chat Events:
- `chat-on-message` - **CRITICAL**: Incoming chat messages, fires for ALL messages including sender's own messages
- `chat-privilege-change` - Chat permission changes (optional)
- `chat-file-upload-progress` - File upload progress (optional)
- `chat-file-download-progress` - File download progress (optional)

**Important**:
- For participant list: Listen to all user events (`user-added`, `user-updated`, `user-removed`) AND media change events (`current-audio-change`, `video-active-change`, `peer-video-state-change`). Call `getAllUser()` in response to refresh the list with the latest state.
- For screen sharing: Listen to all three critical share events (`active-share-change`, `peer-share-state-change`, `passively-stop-share`) to ensure reliable screen share viewing and state management.
- **For chat: The `chat-on-message` event fires for the sender's own messages**. Do NOT manually add sent messages to local state or they will appear twice. Rely entirely on the event for all messages.
