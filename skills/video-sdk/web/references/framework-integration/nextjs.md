# Next.js Integration Guide

Focused patterns for integrating the Zoom Video SDK into a Next.js App Router
app, based on the official
[zoom/videosdk-nextjs-quickstart](https://github.com/zoom/videosdk-nextjs-quickstart)
(`app-router` branch). The quickstart uses the higher-level
**`@zoom/videosdk-react`** hooks/components on top of `@zoom/videosdk`, which
removes most of the manual client/stream wiring.

> **Two ways to integrate, pick one:**
>
> - **`@zoom/videosdk-react` (what the quickstart uses)** — declarative hooks
>   (`useSession`, `useVideoState`, …) and components
>   (`VideoPlayerContainerComponent`, `VideoPlayerComponent`). Least code. This
>   guide documents it.
> - **Raw `@zoom/videosdk`** — full control over the `VideoClient` / `MediaStream`
>   yourself. Follow `react.md` for those patterns; the **Next.js-specific** rules
>   in this guide (client-only boundary, server-side JWT) apply identically.

## Architecture at a Glance

Next.js adds exactly two concerns on top of plain React:

1. **The SDK is browser-only.** It touches `window`/WebAssembly and must never run
   during SSR. Isolate it behind a `dynamic(() => import(...), { ssr: false })`
   boundary.
2. **The JWT is signed on the server.** Generate it in a Server Component (or
   route handler) using a `server-only` module so `ZOOM_SDK_SECRET` never reaches
   the browser, then pass the token down as a prop.

Everything else (rendering video, audio/video controls, participants) is plain
React handled by `@zoom/videosdk-react`.

## 1. Setup

```bash
npm install @zoom/videosdk @zoom/videosdk-react jsrsasign server-only
npm install -D @types/jsrsasign
```

```bash
# .env
ZOOM_SDK_KEY="your-key"
ZOOM_SDK_SECRET="your-secret"
```

The quickstart runs on Next.js 16 (App Router) + React 19 with an empty
`next.config.mjs` — no special build configuration is required.

## 2. Server-side JWT generation

Sign the Video SDK JWT on the server only. Mark the module `server-only` so it can
never be imported into a Client Component by mistake, and read credentials from
env. See `references/auth.md` for the full JWT claim reference.

```typescript
// src/data/getToken.ts
import "server-only";
import { KJUR } from "jsrsasign";

export async function getData(slug: string) {
  return generateSignature(slug, 1); // role 1 = host
}

function generateSignature(sessionName: string, role: number) {
  if (!process.env.ZOOM_SDK_KEY || !process.env.ZOOM_SDK_SECRET) {
    throw new Error("Missing ZOOM_SDK_KEY or ZOOM_SDK_SECRET");
  }
  const iat = Math.round(Date.now() / 1000) - 30;
  const exp = iat + 60 * 60 * 2; // 2 hours
  const oHeader = { alg: "HS256", typ: "JWT" };
  const oPayload = {
    app_key: process.env.ZOOM_SDK_KEY,
    tpc: sessionName,
    role_type: role,
    version: 1,
    iat,
    exp,
  };
  return KJUR.jws.JWS.sign(
    "HS256",
    JSON.stringify(oHeader),
    JSON.stringify(oPayload),
    process.env.ZOOM_SDK_SECRET
  );
}
```

Generate the token in the page's Server Component and hand it to the client
boundary as a prop — no public token endpoint to lock down:

```tsx
// src/app/call/[slug]/page.tsx  (Server Component)
import { getData } from "@/data/getToken";
import VideochatClientWrapper from "@/components/VideochatClientWrapper";

export default async function Page(props: { params: Promise<{ slug: string }> }) {
  const { slug } = await props.params;
  const jwt = await getData(slug);
  return (
    <main className="flex min-h-screen flex-col items-center p-24">
      <VideochatClientWrapper slug={slug} JWT={jwt} />
    </main>
  );
}
```

> **Production note:** anyone who can reach this page gets a host token for that
> session name. Gate the page (or the token generation) behind your own auth and
> derive `role`/`tpc` from the authenticated user — don't trust a client-supplied
> session name blindly.

## 3. The client-only boundary (the key Next.js gotcha)

The SDK must not be evaluated on the server. Put the actual SDK component in a
`"use client"` file, then import it through `next/dynamic` with `ssr: false` from a
thin wrapper. This guarantees the SDK bundle only loads in the browser.

```tsx
// src/components/VideochatClientWrapper.tsx
"use client";
import dynamic from "next/dynamic";

// Loaded dynamically because the Zoom Video SDK needs the browser environment.
const Videochat = dynamic<{ slug: string; JWT: string }>(
  () => import("./Videochat"),
  { ssr: false }
);

export default function VideochatClientWrapper(props: { slug: string; JWT: string }) {
  return <Videochat {...props} />;
}
```

## 4. Session UI with `@zoom/videosdk-react`

`useSession` runs the full init→join→cleanup lifecycle for you and exposes loading
/error/in-session flags. The other hooks read and control state, and the
`VideoPlayer*` components handle `attachVideo`/`detachVideo` automatically — no
refs, no manual element appending.

```tsx
// src/components/Videochat.tsx
"use client";
import { useState, type Dispatch, type SetStateAction } from "react";
import { Mic, MicOff, Video, VideoOff, PhoneOff } from "lucide-react";
import {
  useSession,
  useSessionUsers,
  useVideoState,
  useAudioState,
  VideoPlayerContainerComponent,
  VideoPlayerComponent,
} from "@zoom/videosdk-react";
import { Button } from "./ui/button";

const userName = `User-${Date.now().toString().slice(8)}`;

function Container(props: { slug: string; JWT: string }) {
  const [inCall, setInCall] = useState(false);
  return inCall ? (
    <Videochat {...props} setInCall={setInCall} />
  ) : (
    <Button onClick={() => setInCall(true)}>Join session</Button>
  );
}

function Videochat(props: {
  slug: string;
  JWT: string;
  setInCall: Dispatch<SetStateAction<boolean>>;
}) {
  const { slug: session, JWT, setInCall } = props;

  // Joins on mount, leaves on unmount.
  const { isLoading, isError, isInSession, error } = useSession(session, JWT, userName);
  const participants = useSessionUsers();
  const { isVideoOn, toggleVideo } = useVideoState();
  const { isAudioMuted, toggleMute } = useAudioState();

  if (isLoading) return <div>Loading…</div>;
  if (isError) return <div>Error: {error?.reason}</div>;

  return (
    <div className="flex h-full w-full flex-1 flex-col">
      <h1 className="mb-4 text-center text-3xl font-bold">Session: {session}</h1>

      {isInSession && (
        <VideoPlayerContainerComponent style={{ height: "75vh", borderRadius: 10, overflow: "hidden" }}>
          {participants.map((p) => (
            <VideoPlayerComponent key={p.userId} user={p} />
          ))}
        </VideoPlayerContainerComponent>
      )}

      <div className="mt-4 flex w-[30rem] justify-around self-center rounded-md bg-white p-4">
        <Button onClick={() => void toggleVideo()} title="camera">
          {isVideoOn ? <Video /> : <VideoOff />}
        </Button>
        <Button onClick={toggleMute} title="microphone">
          {isAudioMuted ? <MicOff /> : <Mic />}
        </Button>
        <Button onClick={() => setInCall(false)} title="leave session">
          <PhoneOff />
        </Button>
      </div>
    </div>
  );
}

export default Container;
```

### `@zoom/videosdk-react` reference

Hooks (must be used inside a component mounted under an active `useSession`):

| Hook | Returns |
| --- | --- |
| `useSession(topic, token, userName, password?, idleTimeoutMins?, options?)` | `{ isInSession, isLoading, isError, error }` |
| `useSessionUsers()` | `Participant[]` |
| `useMyself()` | `Participant \| null` |
| `useVideoState()` | `{ isVideoOn, toggleVideo, setVideo }` |
| `useAudioState()` | `{ isAudioMuted, isCapturingAudio, toggleMute, toggleCapture, setMute, setCapture }` |
| `useScreenshare()` | `{ ScreenshareRef, startScreenshare, stopScreenshare, isScreensharing }` |
| `useScreenShareUsers()` | `number[]` (userIds currently sharing) |

Components:

- `VideoPlayerContainerComponent` — wraps the `video-player-container`; give it a
  resolved height. Required parent for the players below.
- `VideoPlayerComponent` — `{ user: Participant; quality?: VideoQuality }`; auto
  attaches/detaches that user's video. Default quality `Video_360P`.
- `ScreenShareContainerComponent` + `ScreenSharePlayerComponent` — receive a
  peer's screen share.
- `LocalScreenShareComponent` — the local screen-share surface; pair with the
  `ScreenshareRef` from `useScreenshare()`.

> `audioOptions`/`videoOptions` join behavior (e.g. join muted) is configured via
> `useSession(..., { disableAudio, disableVideo, audioOptions, videoOptions })`.

## 5. Screen share

```tsx
const { ScreenshareRef, startScreenshare, stopScreenshare, isScreensharing } = useScreenshare();
const sharingUsers = useScreenShareUsers();

return (
  <>
    {/* Local share surface */}
    <LocalScreenShareComponent ref={ScreenshareRef} />
    <Button onClick={() => (isScreensharing ? stopScreenshare() : startScreenshare())}>
      {isScreensharing ? "Stop sharing" : "Share screen"}
    </Button>

    {/* A remote peer's share */}
    <ScreenShareContainerComponent style={{ height: "60vh" }}>
      {sharingUsers.map((userId) => (
        <ScreenSharePlayerComponent key={userId} userId={userId} />
      ))}
    </ScreenShareContainerComponent>
  </>
);
```

## Pages Router

The same two rules apply. Sign the JWT in an API route
(`pages/api/signature.ts`) and load the SDK component with
`dynamic(() => import("../components/Videochat"), { ssr: false })` from the page.
See the quickstart's
[`pages-router`](https://github.com/zoom/videosdk-nextjs-quickstart/tree/pages-router)
branch.

## Next.js-specific Pitfalls

- **Importing `@zoom/videosdk` (or a component that does) into a Server Component
  or shared module** → SSR crash / "window is not defined". Keep all SDK imports
  behind the `dynamic(..., { ssr: false })` boundary.
- **Signing the JWT on the client** → leaks `ZOOM_SDK_SECRET`. Always use a
  `server-only` module / route handler.
- **Open token generation** → treat the page that mints the token as a privileged
  endpoint; add auth and derive the role/topic server-side.
- **No height on `VideoPlayerContainerComponent`** → video attaches but nothing is
  visible. Give the container a resolved height.

## Official references

- Official quickstart: **<https://github.com/zoom/videosdk-nextjs-quickstart>**
  (`app-router` and `pages-router` branches).
- Raw-SDK React patterns (client/stream lifecycle, layouts, annotation):
  `react.md`.
- JWT claims and credentials: `references/auth.md`.
