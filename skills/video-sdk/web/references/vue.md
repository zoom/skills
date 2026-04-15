# Vue Implementation Guide

Complete Vue 3 + Vite 7 + TypeScript + shadcn-vue implementation for Zoom Video SDK.

## Prerequisites

- **Bun** (recommended) or npm/yarn
- **Node.js** 24+
- Basic knowledge of Vue 3 Composition API and TypeScript

## Project Setup

### 1. Create Vite Project

```bash
# Using Bun (recommended)
bun create vite my-zoom-app-vue --template vue-ts
cd my-zoom-app-vue

# Or using npm
npm create vite@latest my-zoom-app-vue -- --template vue-ts
cd my-zoom-app-vue
```

### 2. Install Dependencies

```bash
# Core dependencies
bun add @zoom/videosdk

# Development tools
bun add -D @biomejs/biome

# UI components (shadcn-vue)
bunx shadcn-vue@latest init
bunx shadcn-vue@latest add button card avatar badge dialog input scroll-area

# Initialize Biome for linting/formatting
bunx biome init
```

### 3. Additional UI Icons

```bash
# Install lucide-vue-next for icons (if not already installed by shadcn-vue)
bun add lucide-vue-next
```

## CRITICAL: Vite Configuration

**You MUST configure Vite with COOP/COEP headers** for SharedArrayBuffer support:

```typescript
// vite.config.ts
import vue from "@vitejs/plugin-vue";
import path from "node:path";
import tailwindcss from "@tailwindcss/vite";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [vue(), tailwindcss()],
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

## CRITICAL: TypeScript Custom Elements

```typescript
// src/types/zoom-elements.d.ts
declare module "vue" {
  interface GlobalComponents {
    "video-player-container": DefineComponent<{}, {}, any>;
    "video-player": DefineComponent<{}, {}, any>;
  }
}

export {};
```

## Project Structure

```
src/
├── components/
│   ├── video-session/
│   │   ├── SessionStatus.vue
│   │   ├── VideoGrid.vue
│   │   ├── ParticipantList.vue
│   │   ├── ChatPanel.vue
│   │   ├── ControlBar.vue
│   │   ├── SessionInfo.vue
│   │   └── JoinForm.vue
│   └── ui/
├── composables/
│   ├── useZoomClient.ts
│   ├── useSessionStatus.ts
│   ├── useMediaStream.ts
│   ├── useParticipants.ts
│   ├── useChat.ts
│   └── useActiveSpeaker.ts
├── types/
│   ├── zoom.ts
│   └── zoom-elements.d.ts
└── App.vue
```

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

## Composables

### useZoomClient.ts

```typescript
import ZoomVideo, { type VideoClient } from "@zoom/videosdk";
import { ref, shallowRef } from "vue";

export function useZoomClient() {
  const client = shallowRef<typeof VideoClient | null>(null);
  const isInitialized = ref(false);
  const isJoined = ref(false);

  const getClient = () => {
    if (!client.value) {
      client.value = ZoomVideo.createClient();
    }
    return client.value;
  };

  const init = async () => {
    const zoomClient = getClient();
    await zoomClient.init("en-US", "Global", {
      patchJsMedia: true,
      leaveOnPageUnload: true,
    });
    isInitialized.value = true;

    zoomClient.on("connection-change", (payload) => {
      if (payload.state === "Closed" || payload.state === "Fail") {
        isJoined.value = false;
      }
    });

    return zoomClient;
  };

  const join = async (
    sessionName: string,
    token: string,
    userName: string,
    password?: string
  ) => {
    const zoomClient = getClient();
    if (!isInitialized.value) {
      await init();
    }
    await zoomClient.join(sessionName, token, userName, password);
    isJoined.value = true;
    return zoomClient;
  };

  const leave = async (endSession = false) => {
    const zoomClient = client.value;
    if (zoomClient) {
      await zoomClient.leave(endSession);
      isJoined.value = false;
    }
  };

  const destroy = () => {
    if (client.value) {
      ZoomVideo.destroyClient();
      client.value = null;
      isInitialized.value = false;
      isJoined.value = false;
    }
  };

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
import { ref, type Ref } from "vue";
import type { ConnectionStatus } from "@/types/zoom";

export function useSessionStatus(client: Ref<typeof VideoClient | null>) {
  const status = ref<ConnectionStatus>("Disconnected");

  return { status };
}
```

### useMediaStream.ts

```typescript
import type { Stream, VideoClient } from "@zoom/videosdk";
import { onMounted, ref, shallowRef, watch, type Ref } from "vue";

export function useMediaStream(client: Ref<typeof VideoClient | null>) {
  const stream = shallowRef<typeof Stream | null>(null);
  const audioOn = ref(false);
  const videoOn = ref(false);
  const sharing = ref(false);
  const muted = ref(true);

  watch(client, (newClient) => {
    if (!newClient) return;

    const audioHandler = (payload: { action: string }) => {
      if (payload.action === "join") {
        audioOn.value = true;
        muted.value = true;
      } else if (payload.action === "leave") {
        audioOn.value = false;
      } else if (payload.action === "muted") {
        muted.value = true;
      } else if (payload.action === "unmuted") {
        muted.value = false;
      }
    };

    // CRITICAL: Sync video state with SDK events
    const videoActiveHandler = (payload: { state: string; userId: number }) => {
      const currentUser = newClient.getCurrentUserInfo();
      if (currentUser && payload.userId === currentUser.userId) {
        videoOn.value = payload.state === "Active";
      }
    };

    const shareHandler = (payload: { state: string; userId: number }) => {
      const currentUser = newClient.getCurrentUserInfo();
      if (currentUser && payload.userId === currentUser.userId) {
        sharing.value = payload.state === "Active";
      }
    };

    newClient.on("current-audio-change", audioHandler);
    newClient.on("video-active-change", videoActiveHandler);
    newClient.on("active-share-change", shareHandler);
  });

  const initStream = () => {
    if (client.value) {
      const mediaStream = client.value.getMediaStream();
      stream.value = mediaStream;
      return mediaStream;
    }
    return null;
  };

  const toggleAudio = async () => {
    if (!stream.value) throw new Error("Stream not initialized");

    if (!audioOn.value) {
      await stream.value.startAudio();
    } else if (muted.value) {
      await stream.value.unmuteAudio();
    } else {
      await stream.value.muteAudio();
    }
  };

  const toggleVideo = async () => {
    if (!stream.value) throw new Error("Stream not initialized");

    if (videoOn.value) {
      await stream.value.stopVideo();
      videoOn.value = false;
    } else {
      await stream.value.startVideo();
      videoOn.value = true;
    }
  };

  const toggleScreenShare = async () => {
    if (!stream.value) throw new Error("Stream not initialized");

    if (sharing.value) {
      await stream.value.stopShareScreen();
      sharing.value = false;
    } else {
      await stream.value.startShareScreen();
      sharing.value = true;
    }
  };

  return {
    stream,
    initStream,
    audioOn,
    muted,
    videoOn,
    sharing,
    toggleAudio,
    toggleVideo,
    toggleScreenShare,
  };
}
```

### useParticipants.ts

```typescript
import type { Participant, VideoClient } from "@zoom/videosdk";
import { ref, watch, type Ref } from "vue";

// CRITICAL: Must listen for video state changes to update participant.bVideoOn
export function useParticipants(
  client: Ref<typeof VideoClient | null>,
  isJoined: Ref<boolean>
) {
  const participants = ref<Participant[]>([]);

  const updateList = () => {
    if (client.value) {
      participants.value = client.value.getAllUser();
    }
  };

  watch(client, (newClient) => {
    if (!newClient) return;

    newClient.on("user-added", updateList);
    newClient.on("user-updated", updateList);
    newClient.on("user-removed", updateList);

    // CRITICAL: Listen for video state changes to refresh participant list
    newClient.on("video-active-change", updateList);
    newClient.on("peer-video-state-change", updateList);

    const connectionHandler = (payload: { state: string }) => {
      if (payload.state === "Connected") updateList();
    };
    newClient.on("connection-change", connectionHandler);
  });

  watch(isJoined, (joined) => {
    if (joined && client.value) {
      setTimeout(updateList, 100);
    }
  });

  return participants;
}
```

### useChat.ts

```typescript
import type { VideoClient } from "@zoom/videosdk";
import { ref, watch, type Ref } from "vue";
import type { ChatMessage } from "@/types/zoom";

export function useChat(client: Ref<typeof VideoClient | null>) {
  const messages = ref<ChatMessage[]>([]);

  watch(client, (newClient) => {
    if (!newClient) return;

    const handler = (payload: ChatMessage) => {
      messages.value.push({ ...payload, id: crypto.randomUUID() });
    };
    newClient.on("chat-on-message", handler);
  });

  const sendMessage = async (text: string) => {
    if (!client.value) return;
    const chatClient = client.value.getChatClient();
    await chatClient.send(text);
  };

  const clearMessages = () => {
    messages.value = [];
  };

  return { messages, sendMessage, clearMessages };
}
```

### useActiveSpeaker.ts

```typescript
import type { VideoClient } from "@zoom/videosdk";
import { ref, watch, type Ref } from "vue";

export function useActiveSpeaker(client: Ref<typeof VideoClient | null>) {
  const activeSpeakerId = ref<number | null>(null);

  watch(client, (newClient) => {
    if (!newClient) return;

    const handler = (payload: { activeSpeaker: Array<{ oderId: number }> }) => {
      if (payload.activeSpeaker.length > 0) {
        activeSpeakerId.value = payload.activeSpeaker[0].oderId;
      }
    };
    newClient.on("active-speaker", handler);
  });

  return activeSpeakerId;
}
```

### composables/index.ts

```typescript
export { useZoomClient } from "./useZoomClient";
export { useSessionStatus } from "./useSessionStatus";
export { useMediaStream } from "./useMediaStream";
export { useParticipants } from "./useParticipants";
export { useChat } from "./useChat";
export { useActiveSpeaker } from "./useActiveSpeaker";
```

## Components

### VideoGrid.vue

```vue
<script setup lang="ts">
import type { Participant, Stream } from "@zoom/videosdk";
import { VideoOff } from "lucide-vue-next";
import { onBeforeUnmount, onMounted, ref, watch, type PropType } from "vue";
import { Avatar, AvatarFallback } from "@/components/ui/avatar";
import { Badge } from "@/components/ui/badge";
import type { ViewMode } from "@/types/zoom";

const props = defineProps({
  participants: {
    type: Array as PropType<Participant[]>,
    required: true,
  },
  viewMode: {
    type: String as PropType<ViewMode>,
    required: true,
  },
  activeSpeakerId: {
    type: Number as PropType<number | null>,
    default: null,
  },
  stream: {
    type: Object as PropType<typeof Stream | null>,
    default: null,
  },
  currentUserId: {
    type: Number,
    default: undefined,
  },
  currentUserVideoOn: {
    type: Boolean,
    default: false,
  },
});

// VideoTile Component Logic
interface VideoTileProps {
  participant: Participant;
  stream: typeof Stream | null;
  isLarge?: boolean;
  isSpeaking?: boolean;
  currentUserId?: number;
  currentUserVideoOn?: boolean;
}

function useVideoTile(props: VideoTileProps) {
  const containerRef = ref<HTMLElement | null>(null);

  const isCurrentUser = props.currentUserId === props.participant.userId;
  const isVideoOn = isCurrentUser
    ? props.currentUserVideoOn ?? props.participant.bVideoOn
    : props.participant.bVideoOn;

  const attachVideo = async () => {
    if (!props.stream || !containerRef.value) return;

    const container = containerRef.value;

    if (!isVideoOn) {
      try {
        props.stream.detachVideo(props.participant.userId);
        container.innerHTML = "";
      } catch {
        // Ignore detach errors
      }
      return;
    }

    try {
      const videoElement = await props.stream.attachVideo(
        props.participant.userId,
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

  onMounted(() => {
    attachVideo();
  });

  watch(() => isVideoOn, attachVideo);

  onBeforeUnmount(() => {
    if (props.stream) {
      try {
        props.stream.detachVideo(props.participant.userId);
      } catch {
        // Ignore detach errors
      }
    }
  });

  return { containerRef, isVideoOn };
}

const getGridCols = (count: number) => {
  if (count === 1) return "grid-cols-1";
  if (count <= 4) return "grid-cols-2";
  if (count <= 9) return "grid-cols-3";
  return "grid-cols-4";
};
</script>

<template>
  <div
    v-if="participants.length === 0"
    class="flex items-center justify-center h-full"
  >
    <p class="text-muted-foreground">Waiting for participants...</p>
  </div>

  <!-- Speaker View -->
  <div v-else-if="viewMode === 'speaker'" class="relative h-full p-4">
    <VideoTile
      :participant="
        participants.find((p) => p.userId === activeSpeakerId) ||
        participants[0]
      "
      :stream="stream"
      :is-large="true"
      :is-speaking="
        (
          participants.find((p) => p.userId === activeSpeakerId) ||
          participants[0]
        ).userId === activeSpeakerId
      "
      :current-user-id="currentUserId"
      :current-user-video-on="currentUserVideoOn"
    />
    <div
      v-if="
        participants.filter(
          (p) =>
            p.userId !==
            (
              participants.find((p) => p.userId === activeSpeakerId) ||
              participants[0]
            ).userId
        ).length > 0
      "
      class="absolute bottom-6 left-6 flex gap-2"
    >
      <div
        v-for="p in participants
          .filter(
            (p) =>
              p.userId !==
              (
                participants.find((p) => p.userId === activeSpeakerId) ||
                participants[0]
              ).userId
          )
          .slice(0, 4)"
        :key="p.userId"
        class="w-32"
      >
        <VideoTile
          :participant="p"
          :stream="stream"
          :is-speaking="p.userId === activeSpeakerId"
          :current-user-id="currentUserId"
          :current-user-video-on="currentUserVideoOn"
        />
      </div>
    </div>
  </div>

  <!-- Gallery View -->
  <div
    v-else
    :class="`grid ${getGridCols(participants.length)} gap-2 h-full p-4`"
  >
    <VideoTile
      v-for="p in participants"
      :key="p.userId"
      :participant="p"
      :stream="stream"
      :is-speaking="p.userId === activeSpeakerId"
      :current-user-id="currentUserId"
      :current-user-video-on="currentUserVideoOn"
    />
  </div>
</template>

<!-- VideoTile Component -->
<script setup lang="ts">
const tileProps = defineProps({
  participant: {
    type: Object as PropType<Participant>,
    required: true,
  },
  stream: {
    type: Object as PropType<typeof Stream | null>,
    default: null,
  },
  isLarge: {
    type: Boolean,
    default: false,
  },
  isSpeaking: {
    type: Boolean,
    default: false,
  },
  currentUserId: {
    type: Number,
    default: undefined,
  },
  currentUserVideoOn: {
    type: Boolean,
    default: false,
  },
});

const { containerRef, isVideoOn } = useVideoTile(tileProps);
</script>

<template>
  <div
    :class="[
      'relative bg-muted rounded-lg overflow-hidden flex items-center justify-center',
      isLarge ? 'w-full h-full' : 'aspect-video',
      isSpeaking ? 'ring-2 ring-green-500' : '',
    ]"
  >
    <!-- CRITICAL: Must use video-player-container custom element -->
    <video-player-container
      ref="containerRef"
      class="w-full h-full"
      :style="{ minHeight: '100px', display: isVideoOn ? 'block' : 'none' }"
    />

    <div
      v-if="!isVideoOn"
      class="flex flex-col items-center justify-center gap-2"
    >
      <Avatar :class="isLarge ? 'h-24 w-24' : 'h-16 w-16'">
        <AvatarFallback :class="isLarge ? 'text-3xl' : 'text-xl'">
          {{ participant.displayName.charAt(0).toUpperCase() }}
        </AvatarFallback>
      </Avatar>
      <VideoOff class="h-5 w-5 text-muted-foreground" />
    </div>

    <div
      class="absolute bottom-2 left-2 right-2 flex items-center justify-between"
    >
      <Badge variant="secondary" class="text-xs truncate max-w-[80%]">
        {{ participant.displayName }}
        {{ participant.isHost ? " (Host)" : "" }}
      </Badge>
    </div>
  </div>
</template>
```

### SessionStatus.vue

```vue
<script setup lang="ts">
import { computed, type PropType } from "vue";
import { Badge } from "@/components/ui/badge";
import type { ConnectionStatus } from "@/types/zoom";

const props = defineProps({
  status: {
    type: String as PropType<ConnectionStatus>,
    required: true,
  },
});

const variants: Record<ConnectionStatus, string> = {
  Connected: "bg-green-500 text-white",
  Connecting: "bg-yellow-500 animate-pulse",
  Reconnecting: "bg-yellow-500 animate-pulse",
  Disconnected: "bg-gray-500",
  Failed: "bg-red-500 text-white",
};

const badgeClass = computed(() => variants[props.status]);
</script>

<template>
  <Badge :class="badgeClass">{{ status }}</Badge>
</template>
```

### ControlBar.vue

```vue
<script setup lang="ts">
import {
  Info,
  LogOut,
  MessageSquare,
  Mic,
  MicOff,
  Monitor,
  Users,
  Video,
  VideoOff,
} from "lucide-vue-next";
import { Button } from "@/components/ui/button";

defineProps({
  audioOn: { type: Boolean, required: true },
  muted: { type: Boolean, required: true },
  videoOn: { type: Boolean, required: true },
  sharing: { type: Boolean, required: true },
  showChat: { type: Boolean, required: true },
  showParticipants: { type: Boolean, required: true },
});

const emit = defineEmits([
  "toggleAudio",
  "toggleVideo",
  "toggleShare",
  "leave",
  "toggleChat",
  "toggleParticipants",
  "toggleInfo",
]);
</script>

<template>
  <div
    class="flex items-center justify-center gap-2 p-4 border-t bg-background"
  >
    <Button
      :variant="audioOn && !muted ? 'default' : 'secondary'"
      size="icon"
      @click="emit('toggleAudio')"
    >
      <MicOff v-if="!audioOn || muted" class="h-5 w-5" />
      <Mic v-else class="h-5 w-5" />
    </Button>

    <Button
      :variant="videoOn ? 'default' : 'secondary'"
      size="icon"
      @click="emit('toggleVideo')"
    >
      <Video v-if="videoOn" class="h-5 w-5" />
      <VideoOff v-else class="h-5 w-5" />
    </Button>

    <Button
      :variant="sharing ? 'default' : 'secondary'"
      size="icon"
      @click="emit('toggleShare')"
    >
      <Monitor class="h-5 w-5" />
    </Button>

    <div class="w-px h-8 bg-border mx-2" />

    <Button
      :variant="showParticipants ? 'default' : 'ghost'"
      size="icon"
      @click="emit('toggleParticipants')"
    >
      <Users class="h-5 w-5" />
    </Button>

    <Button
      :variant="showChat ? 'default' : 'ghost'"
      size="icon"
      @click="emit('toggleChat')"
    >
      <MessageSquare class="h-5 w-5" />
    </Button>

    <Button variant="ghost" size="icon" @click="emit('toggleInfo')">
      <Info class="h-5 w-5" />
    </Button>

    <div class="w-px h-8 bg-border mx-2" />

    <Button variant="destructive" @click="emit('leave')">
      <LogOut class="h-4 w-4 mr-2" />
      Leave
    </Button>
  </div>
</template>
```

### ParticipantList.vue

```vue
<script setup lang="ts">
import type { Participant } from "@zoom/videosdk";
import { Mic, MicOff, Video } from "lucide-vue-next";
import type { PropType } from "vue";
import { Avatar, AvatarFallback } from "@/components/ui/avatar";
import { Badge } from "@/components/ui/badge";
import { ScrollArea } from "@/components/ui/scroll-area";

defineProps({
  participants: {
    type: Array as PropType<Participant[]>,
    required: true,
  },
});
</script>

<template>
  <div class="flex flex-col h-full">
    <div class="p-3 border-b">
      <h3 class="font-semibold">Participants ({{ participants.length }})</h3>
    </div>
    <ScrollArea class="flex-1">
      <div class="p-2 space-y-1">
        <div
          v-for="user in participants"
          :key="user.userId"
          class="flex items-center gap-2 p-2 rounded-md hover:bg-muted"
        >
          <Avatar class="h-8 w-8">
            <AvatarFallback>{{ user.displayName[0] }}</AvatarFallback>
          </Avatar>
          <span class="flex-1 text-sm truncate">{{ user.displayName }}</span>
          <Badge v-if="user.isHost" variant="secondary" class="text-xs"
            >Host</Badge
          >
          <Video v-if="user.bVideoOn" class="h-4 w-4 text-muted-foreground" />
          <MicOff v-if="user.muted" class="h-4 w-4 text-muted-foreground" />
          <Mic v-else class="h-4 w-4 text-muted-foreground" />
        </div>
      </div>
    </ScrollArea>
  </div>
</template>
```

### ChatPanel.vue

```vue
<script setup lang="ts">
import { Send } from "lucide-vue-next";
import { ref, type PropType } from "vue";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { ScrollArea } from "@/components/ui/scroll-area";
import type { ChatMessage } from "@/types/zoom";

defineProps({
  messages: {
    type: Array as PropType<ChatMessage[]>,
    required: true,
  },
});

const emit = defineEmits(["send"]);

const input = ref("");

const handleSubmit = (e: Event) => {
  e.preventDefault();
  if (input.value.trim()) {
    emit("send", input.value.trim());
    input.value = "";
  }
};
</script>

<template>
  <div class="flex flex-col h-full">
    <div class="p-3 border-b">
      <h3 class="font-semibold">Chat</h3>
    </div>
    <ScrollArea class="flex-1 p-3">
      <div class="space-y-3">
        <div v-for="msg in messages" :key="msg.id" class="text-sm">
          <span class="font-medium">{{ msg.sender.name }}: </span>
          <span>{{ msg.message }}</span>
        </div>
      </div>
    </ScrollArea>
    <form @submit="handleSubmit" class="p-3 border-t flex gap-2">
      <Input v-model="input" placeholder="Type a message..." class="flex-1" />
      <Button type="submit" size="icon">
        <Send class="h-4 w-4" />
      </Button>
    </form>
  </div>
</template>
```

### JoinForm.vue

```vue
<script setup lang="ts">
import { Loader2, Video } from "lucide-vue-next";
import { onMounted, ref } from "vue";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";

defineProps({
  isLoading: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(["join"]);

const sessionName = ref("example-session");
const userName = ref("example-user");
const token = ref("");
const isFetchingToken = ref(false);
const tokenError = ref<string | null>(null);

const fetchToken = async (session: string) => {
  if (!session) return;

  isFetchingToken.value = true;
  tokenError.value = null;

  try {
    const response = await fetch("http://localhost:4000", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ sessionName: session, role: 1 }),
    });

    if (!response.ok) throw new Error("Failed to get token");

    const data = await response.json();
    token.value = data.signature;
  } catch (error) {
    tokenError.value = "Failed to get token. Make sure your server is running.";
  } finally {
    isFetchingToken.value = false;
  }
};

onMounted(() => {
  if (sessionName.value) fetchToken(sessionName.value);
});

const handleSubmit = (e: Event) => {
  e.preventDefault();
  if (sessionName.value && userName.value && token.value) {
    emit("join", sessionName.value, userName.value, token.value);
  }
};
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-background p-4">
    <Card class="w-full max-w-md">
      <CardHeader class="text-center">
        <div
          class="mx-auto mb-4 h-12 w-12 rounded-full bg-primary/10 flex items-center justify-center"
        >
          <Video class="h-6 w-6 text-primary" />
        </div>
        <CardTitle class="text-2xl">Join Video Session</CardTitle>
      </CardHeader>
      <CardContent>
        <form @submit="handleSubmit" class="space-y-4">
          <div class="space-y-2">
            <label class="text-sm font-medium">Session Name</label>
            <Input
              v-model="sessionName"
              placeholder="Enter session name"
              required
            />
          </div>
          <div class="space-y-2">
            <label class="text-sm font-medium">Your Name</label>
            <Input v-model="userName" placeholder="Enter your name" required />
          </div>
          <div class="space-y-2">
            <label class="text-sm font-medium">JWT Token</label>
            <div class="flex gap-2">
              <Input
                v-model="token"
                :placeholder="isFetchingToken ? 'Fetching...' : 'Token'"
                required
                class="flex-1"
              />
              <Button
                type="button"
                variant="secondary"
                @click="() => fetchToken(sessionName)"
                :disabled="isFetchingToken || !sessionName"
              >
                <Loader2 v-if="isFetchingToken" class="h-4 w-4 animate-spin" />
                <span v-else>Refresh</span>
              </Button>
            </div>
            <p v-if="tokenError" class="text-xs text-destructive">
              {{ tokenError }}
            </p>
          </div>
          <Button type="submit" class="w-full" :disabled="isLoading || !token">
            {{ isLoading ? "Joining..." : "Join Session" }}
          </Button>
        </form>
      </CardContent>
    </Card>
  </div>
</template>
```

### SessionInfo.vue

```vue
<script setup lang="ts">
import type { VideoClient } from "@zoom/videosdk";
import type { PropType } from "vue";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";

const props = defineProps({
  client: {
    type: Object as PropType<typeof VideoClient | null>,
    default: null,
  },
  open: {
    type: Boolean,
    required: true,
  },
});

const emit = defineEmits(["update:open"]);

const sessionInfo = props.client?.getSessionInfo();
const currentUser = props.client?.getCurrentUserInfo();
</script>

<template>
  <Dialog :open="open" @update:open="emit('update:open', $event)">
    <DialogContent>
      <DialogHeader>
        <DialogTitle>Session Information</DialogTitle>
      </DialogHeader>
      <div class="grid gap-2 text-sm">
        <div>
          <span class="font-medium">Session:</span> {{ sessionInfo?.topic }}
        </div>
        <div>
          <span class="font-medium">Session ID:</span>
          {{ sessionInfo?.sessionId }}
        </div>
        <div>
          <span class="font-medium">Your Name:</span>
          {{ currentUser?.displayName }}
        </div>
        <div>
          <span class="font-medium">Your Role:</span>
          {{
            currentUser?.isHost
              ? "Host"
              : currentUser?.isManager
              ? "Manager"
              : "Participant"
          }}
        </div>
        <div>
          <span class="font-medium">Participants:</span>
          {{ client?.getAllUser().length }}
        </div>
      </div>
    </DialogContent>
  </Dialog>
</template>
```

### SessionPage.vue (Main Page)

```vue
<script setup lang="ts">
import { Grid2X2, LayoutGrid } from "lucide-vue-next";
import { onMounted, onUnmounted, ref, watch } from "vue";
import { Button } from "@/components/ui/button";
import {
  useActiveSpeaker,
  useChat,
  useMediaStream,
  useParticipants,
  useSessionStatus,
  useZoomClient,
} from "@/composables";
import type { ViewMode } from "@/types/zoom";
import ChatPanel from "./ChatPanel.vue";
import ControlBar from "./ControlBar.vue";
import JoinForm from "./JoinForm.vue";
import ParticipantList from "./ParticipantList.vue";
import SessionInfo from "./SessionInfo.vue";
import SessionStatus from "./SessionStatus.vue";
import VideoGrid from "./VideoGrid.vue";

const { client, getClient, join, leave, destroy, isJoined } = useZoomClient();
const { status } = useSessionStatus(client);
const {
  stream,
  initStream,
  audioOn,
  muted,
  videoOn,
  sharing,
  toggleAudio,
  toggleVideo,
  toggleScreenShare,
} = useMediaStream(client);
const participants = useParticipants(client, isJoined);
const activeSpeakerId = useActiveSpeaker(client);
const { messages, sendMessage, clearMessages } = useChat(client);

const viewMode = ref<ViewMode>("gallery");
const showChat = ref(false);
const showParticipants = ref(false);
const showInfo = ref(false);
const error = ref<string | null>(null);
const isLoading = ref(false);

const currentUser = computed(() => client.value?.getCurrentUserInfo());

watch([isJoined, client, stream], ([joined, clientVal, streamVal]) => {
  if (joined && clientVal && !streamVal) {
    initStream();
  }
});

const handleJoin = async (
  sessionName: string,
  userName: string,
  token: string
) => {
  try {
    isLoading.value = true;
    error.value = null;
    status.value = "Connecting";
    await join(sessionName, token, userName);
    status.value = "Connected";
  } catch (err) {
    error.value = err instanceof Error ? err.message : "Failed to join session";
    status.value = "Failed";
  } finally {
    isLoading.value = false;
  }
};

const handleLeave = async () => {
  clearMessages();
  await leave();
};

const handleToggleAudio = async () => {
  try {
    error.value = null;
    await toggleAudio();
  } catch (err) {
    error.value = err instanceof Error ? err.message : "Audio error";
    setTimeout(() => (error.value = null), 3000);
  }
};

const handleToggleVideo = async () => {
  try {
    error.value = null;
    await toggleVideo();
  } catch (err) {
    error.value = err instanceof Error ? err.message : "Video error";
    setTimeout(() => (error.value = null), 3000);
  }
};

const handleToggleScreenShare = async () => {
  try {
    error.value = null;
    await toggleScreenShare();
  } catch (err) {
    error.value = err instanceof Error ? err.message : "Screen share error";
    setTimeout(() => (error.value = null), 3000);
  }
};

onMounted(() => {
  getClient();
});

onUnmounted(() => {
  destroy();
});
</script>

<template>
  <div v-if="!isJoined">
    <JoinForm :is-loading="isLoading" @join="handleJoin" />
    <div
      v-if="error"
      class="fixed bottom-4 left-1/2 -translate-x-1/2 bg-destructive text-white px-4 py-2 rounded-md"
    >
      {{ error }}
    </div>
  </div>

  <div v-else class="flex flex-col h-screen bg-background">
    <header class="flex items-center justify-between px-4 py-2 border-b">
      <SessionStatus :status="status" />
      <div class="flex gap-1">
        <Button
          :variant="viewMode === 'speaker' ? 'default' : 'ghost'"
          size="sm"
          @click="viewMode = 'speaker'"
        >
          <LayoutGrid class="h-4 w-4 mr-1" />
          Speaker
        </Button>
        <Button
          :variant="viewMode === 'gallery' ? 'default' : 'ghost'"
          size="sm"
          @click="viewMode = 'gallery'"
        >
          <Grid2X2 class="h-4 w-4 mr-1" />
          Gallery
        </Button>
      </div>
    </header>

    <div class="flex flex-1 overflow-hidden">
      <main class="flex-1 overflow-hidden">
        <!-- CRITICAL: Pass videoOn and currentUserId to VideoGrid -->
        <VideoGrid
          :participants="participants"
          :view-mode="viewMode"
          :active-speaker-id="activeSpeakerId"
          :stream="stream"
          :current-user-id="currentUser?.userId"
          :current-user-video-on="videoOn"
        />
      </main>

      <aside v-if="showParticipants" class="w-64 border-l overflow-hidden">
        <ParticipantList :participants="participants" />
      </aside>

      <aside v-if="showChat" class="w-80 border-l overflow-hidden">
        <ChatPanel :messages="messages" @send="sendMessage" />
      </aside>
    </div>

    <ControlBar
      :audio-on="audioOn"
      :muted="muted"
      :video-on="videoOn"
      :sharing="sharing"
      :show-chat="showChat"
      :show-participants="showParticipants"
      @toggle-audio="handleToggleAudio"
      @toggle-video="handleToggleVideo"
      @toggle-share="handleToggleScreenShare"
      @leave="handleLeave"
      @toggle-chat="showChat = !showChat"
      @toggle-participants="showParticipants = !showParticipants"
      @toggle-info="showInfo = true"
    />

    <SessionInfo
      :client="client"
      :open="showInfo"
      @update:open="showInfo = $event"
    />

    <div
      v-if="error"
      class="fixed bottom-20 left-1/2 -translate-x-1/2 bg-destructive text-white px-4 py-2 rounded-md z-50"
    >
      {{ error }}
    </div>
  </div>
</template>
```

## App.vue

```vue
<script setup lang="ts">
import SessionPage from "@/components/video-session/SessionPage.vue";
</script>

<template>
  <SessionPage />
</template>
```

## Key Differences from React

1. **Composables vs Hooks**: Vue uses composables (similar to React hooks) but with different reactivity patterns
2. **Refs**: Vue uses `ref()` and `shallowRef()` for reactive state
3. **Watch**: Vue's `watch()` replaces React's `useEffect()` for side effects
4. **Template Syntax**: Vue uses template-based syntax instead of JSX
5. **Event Handling**: Vue uses `@event` syntax and `emit()` for component communication
6. **Lifecycle**: Vue uses `onMounted()`, `onUnmounted()`, etc. instead of `useEffect()`

## Best Practices

1. Use `shallowRef()` for complex objects like VideoClient and Stream
2. Use `watch()` to listen for client changes and attach event listeners
3. Clean up event listeners in component unmount
4. Use TypeScript for type safety
5. Follow Vue 3 Composition API patterns
6. Use `computed()` for derived state
7. Emit events for parent-child communication
