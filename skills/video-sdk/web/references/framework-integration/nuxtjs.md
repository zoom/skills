# Nuxt Integration Guide

Focused patterns for integrating the Zoom Video SDK into a Nuxt app, based on the
official
[zoom/videosdk-vue-nuxt-quickstart](https://github.com/zoom/videosdk-vue-nuxt-quickstart)
(Nuxt 4 + Vue 3). Unlike the Next.js quickstart, this one uses the **raw
`@zoom/videosdk`** directly — there is no Nuxt-specific wrapper package.

> **Read `vue.md` for the canonical SDK patterns.** Nuxt is Vue 3 under the hood,
> so the reactive SDK wiring — holding one `createClient()` instance, the
> init→join lifecycle, `connection-change` / `ConnectionState` handling,
> participant sync, the declarative `video-player` attach API, screen share,
> annotation — is documented in `vue.md` and applies unchanged. This guide covers
> only the **Nuxt-specific** concerns layered on top.

## Architecture at a Glance

Nuxt adds three concerns on top of a plain Vue SPA:

1. **The SDK is browser-only.** It touches `window`/WebAssembly and must never run
   during SSR. Wrap the SDK component in Nuxt's built-in `<ClientOnly>`.
2. **The JWT is signed on the server.** Generate it in a Nuxt server route using
   `useRuntimeConfig()` so `ZOOM_SDK_SECRET` never reaches the browser, then fetch
   it with `useFetch`.
3. **Custom elements must be registered with the Vue compiler.** Declare
   `video-player-container` / `video-player` via `vue.compilerOptions.isCustomElement`
   in `nuxt.config.ts`, otherwise Nuxt treats them as Vue components and warns/fails.

## 1. Setup

```bash
npm install @zoom/videosdk jsrsasign
npm install -D @types/jsrsasign
```

```bash
# .env
ZOOM_SDK_KEY="your-key"
ZOOM_SDK_SECRET="your-secret"
```

Configure `nuxt.config.ts` — expose the credentials to the **server only** via
`runtimeConfig`, and register the custom elements:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Private runtime config — available on the server only (never shipped to the client).
  runtimeConfig: {
    ZoomVideoSDKKey: process.env.ZOOM_SDK_KEY,
    ZoomVideoSDKSecret: process.env.ZOOM_SDK_SECRET,
  },

  // Tell the Vue compiler these are custom elements, not Vue components.
  vue: {
    compilerOptions: {
      isCustomElement: (tag) =>
        tag === "video-player-container" || tag === "video-player",
    },
  },

  compatibilityDate: "2025-07-07",
});
```

> The quickstart only registers `video-player-container` because it appends the
> `VideoPlayer` element returned by `attachVideo` imperatively. If you follow the
> **declarative** pattern from `vue.md` (rendering `<video-player>` in the
> template), you must also register `video-player` as shown above.

## 2. Server-side JWT generation

Sign the Video SDK JWT in a Nuxt server route. Read credentials from
`useRuntimeConfig()` (server-only), never from the client. See `references/auth.md`
for the full JWT claim reference.

```typescript
// server/api/token.ts
import { KJUR } from "jsrsasign";

export default defineEventHandler(async (event) => {
  const query = getQuery(event);
  if (typeof query.slug !== "string") {
    throw createError({ statusCode: 400, statusMessage: "Add a session name as string" });
  }
  return generateSignature(query.slug, 1); // role 1 = host
});

function generateSignature(sessionName: string, role: number) {
  const { ZoomVideoSDKKey: sdkKey, ZoomVideoSDKSecret: sdkSecret } = useRuntimeConfig();
  if (!sdkKey || !sdkSecret) {
    throw new Error("Missing ZOOM_SDK_KEY or ZOOM_SDK_SECRET");
  }
  const iat = Math.round(Date.now() / 1000) - 30;
  const exp = iat + 60 * 60 * 2; // 2 hours
  const oHeader = { alg: "HS256", typ: "JWT" };
  const oPayload = {
    app_key: sdkKey,
    tpc: sessionName,
    role_type: role,
    version: 1,
    iat,
    exp,
  };
  return KJUR.jws.JWS.sign("HS256", JSON.stringify(oHeader), JSON.stringify(oPayload), sdkSecret);
}
```

> **Production note:** anyone who can hit `/api/token?slug=...` gets a host token
> for that session name. Gate the route behind your own auth and derive
> `role`/`tpc` from the authenticated user — don't trust a client-supplied session
> name blindly.

## 3. The client-only boundary (the key Nuxt gotcha)

Fetch the token with `useFetch` (runs on the server during SSR) and render the SDK
component inside `<ClientOnly>` so it only mounts in the browser:

```vue
<!-- pages/call/[slug].vue -->
<script setup lang="ts">
const route = useRoute();
const { data: JWT } = await useFetch(`/api/token?slug=${route.params.slug}`);
</script>

<template>
  <div class="flex flex-col">
    <h1 class="my-4 text-center text-3xl font-bold">Session: {{ route.params.slug }}</h1>
    <ClientOnly>
      <Videocall :slug="route.params.slug" :JWT="JWT" />
    </ClientOnly>
  </div>
</template>
```

```vue
<!-- pages/index.vue -->
<script setup lang="ts">
const sessionName = ref("");
const create = async () => {
  if (sessionName.value) await navigateTo(`/call/${sessionName.value}`);
};
</script>

<template>
  <input v-model="sessionName" type="text" placeholder="Session Name" />
  <button @click="create">Create</button>
</template>
```

> `<ClientOnly>` is the Nuxt equivalent of Next.js's `dynamic(..., { ssr: false })`.
> An alternative is naming the component `Videocall.client.vue`, which makes Nuxt
> render it on the client only without an explicit wrapper.

## 4. The session component

`components/Videocall.vue` is auto-imported by Nuxt and owns the SDK lifecycle. It
receives the topic (`slug`) and `JWT` as props. The example below mirrors the
quickstart's minimal flow; for the **canonical reactive patterns** — participant
synchronization, `connection-change` handling, and declarative `<video-player>`
rendering — follow `vue.md` (they work identically inside a Nuxt component).

```vue
<!-- components/Videocall.vue -->
<script setup lang="ts">
import ZoomVideo, { type VideoPlayer, VideoQuality } from "@zoom/videosdk";

const props = defineProps<{ slug: string; JWT: string }>();
const userName = `User-${String(Date.now()).slice(6)}`;
const client = ZoomVideo.createClient();

const videoContainer = ref<HTMLElement | null>(null);
const inSession = ref(false);

onMounted(() => client.init("en-US", "Global", { patchJsMedia: true }));
onBeforeUnmount(() => ZoomVideo.destroyClient());

const startCall = async () => {
  client.on("peer-video-state-change", renderVideo);
  await client.join(props.slug, props.JWT, userName);
  const stream = client.getMediaStream();
  await stream.startAudio();
  await stream.startVideo();
  await renderVideo({ action: "Start", userId: client.getCurrentUserInfo().userId });
  inSession.value = true;
};

const renderVideo = async (e: { action: "Start" | "Stop"; userId: number }) => {
  const stream = client.getMediaStream();
  if (e.action === "Stop") {
    const el = await stream.detachVideo(e.userId);
    (Array.isArray(el) ? el : [el]).forEach((n) => n.remove());
  } else {
    const el = await stream.attachVideo(e.userId, VideoQuality.Video_360P);
    videoContainer.value?.appendChild(el as VideoPlayer);
  }
};
</script>

<template>
  <div v-show="inSession" class="h-[80vh] w-[80vw] self-center overflow-hidden">
    <video-player-container ref="videoContainer"></video-player-container>
  </div>
  <button v-if="!inSession" @click="startCall">Join</button>
</template>
```

> The quickstart appends the element returned by `attachVideo` imperatively. The
> recommended approach (declarative `<video-player>` keyed by participant, with
> `attachVideo(userId, quality, element)`) is in `vue.md` §4 — prefer it for real
> apps; just remember to register `video-player` in `isCustomElement` (step 1).

## Nuxt-specific Pitfalls

- **Rendering the SDK component without `<ClientOnly>`** (or a `.client.vue`
  suffix) → SSR crash / "window is not defined". The SDK must run client-side.
- **Forgetting `isCustomElement`** → Nuxt logs "Failed to resolve component:
  video-player-container" and the video surface never appears.
- **Putting credentials in `runtimeConfig.public`** → leaks `ZOOM_SDK_SECRET` to
  the browser. Keep them in the top-level (private) `runtimeConfig`.
- **Open token route** → treat `/api/token` as a privileged endpoint; add auth and
  derive the role/topic server-side.
- **No height on `video-player-container`** → video attaches but nothing is
  visible. Give the container a resolved height.

## Official references

- Official quickstart: **<https://github.com/zoom/videosdk-vue-nuxt-quickstart>**
  (`main` and `nuxt4` branches).
- Canonical Vue SDK patterns (lifecycle, participant sync, declarative video,
  screen share, annotation): `vue.md`.
- JWT claims and credentials: `references/auth.md`.
