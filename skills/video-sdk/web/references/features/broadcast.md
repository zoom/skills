<!-- Source: https://developers.zoom.us/docs/video-sdk/web/broadcast.md -->
<!-- Fetched: 2026-06-15 -->

# Broadcast streaming


The broadcast streaming feature enables up to 10 million concurrent viewers with latency as low as 200ms, and also supports a rewind mode. It helps session hosts extend the reach of their content to a significantly larger audience. _**As a best practice, we do not recommend that you host a live broadcast session and a Video SDK session on the same page without closing one first.**_

> [Contact Sales](https://explore.zoom.us/en/video-sdk/#sf_form) to enable broadcast streaming.

## For Session Host

Perform the following actions for the session host.

### 1. Initialize the Broadcast Streaming Client

After joining a session, call `client.getBroadcastStreamingClient()` to retrieve the broadcast streaming client instance.

```typescript
const broadcastStreaming = client.getBroadcastStreamingClient();
```

### 2. Check Broadcast Streaming Availability

Check whether broadcast streaming is enabled for the current account.

```javascript
const isBroadcastStreamingEnabled =
    broadcastStreaming.isBroadcastStreamingEnable();
```

### 3. Start Broadcast Streaming

During a session, start video, audio, and screen sharing as usual. When you're ready to initiate broadcast streaming, call `startBroadcast`.

```javascript
const channelId = await broadcastStreaming.startBroadcast();
// Warning: Make sure to securely manage the channelId, as it is required for attendees to watch the stream.
```

It may take a few seconds to start broadcast streaming. To track the status changes, listen to the `broadcast-streaming-status` event.

```javascript
client.on("broadcast-streaming-status", (payload) => {
    const { status } = payload;
    console.log(`broadcast streaming status changed to: ${status}`);
    // Add any additional handling logic here
});
```

### 4. Stop Broadcast Streaming.

To stop broadcast streaming manually, use `stopBroadcast`.

```javascript
await broadcastStreaming.stopBroadcast();
```

If you intend to stream until the session ends, you do not need to call this method as the broadcast server will automatically stop streaming at session termination.

## For Attendees

Unlike regular session users, broadcast streaming viewers do not join the video session. Therefore, the implementation flow is different.

### 1. Import ZoomStreaming

Import using npm or CDN.

```javascript
import ZoomStreaming from "@zoom/videosdk/broadcast-streaming";
```

If you're using TypeScript and encounter the error.

```javascript
Cannot find module '@zoom/videosdk/broadcast-streaming' or its corresponding type declarations.
```

You can resolve it by setting the `moduleResolution` to `"bundler"` in your `tsconfig.json` or use the full path for broadcast-streaming: `@zoom/videosdk/dist/broadcast-streaming`.

Add the following script and replace `{version}` with the current version of the Video SDK.

```html
<script src="https://source.zoom.us/videosdk/zoom-streaming-{version}.min.js"></script>
```

Then define `ZoomStreaming` in your component file.

```javascript
const ZoomStreaming = window.WebBroadcastStreaming.default;
```

### 2. Initialize the Streaming Client.

After importing, create a streaming client instance.

```javascript
const streaming = ZoomStreaming.createClient();
// If using a private deployment, specify custom asset paths via dependentAssets, consistent with client.init() configuration.
```

### 3. Attach/Detach Streaming.

Zoom provides a DOM-based video attach mechanism that allows you to render the stream in specific HTML elements.

```html
<live-video-container></live-video-container>
```

Adjust the CSS to your liking.

```css
live-video-container {
    width: 50%;
    height: auto;
    aspect-ratio: 16/9;
}
live-video {
    width: 100%;
    aspect-ratio: 16/9;
}
```

#### Attach Streaming

Use `attachStreaming` to create a `live-video` element. You'll need the channel ID and a valid JWT token from the same account as the host.

```javascript
const element = await streaming.attachStreaming(
    channelId,
    JWT_TOKEN,
    VideoQuality.Video_720P,
);
document.querySelector("live-video-container")?.appendChild(element);
// Enable built-in controls if needed
element.setAttribute("controls", "true");
```

The streaming default mode is live. To enable rewind mode (HLS), set the rewind attribute.

```javascript
element.setAttribute("rewind", "true");
```

#### Handle Audio Auto-play Restrictions

Due to browser autoplay restrictions, if audio is attempted before user interaction, playback might fail. To handle this, listen for the `auto-play-audio-failed` event.

```javascript
streaming.on("auto-play-audio-failed", () => {
    // promote the user to interact with the page
    console.log(
        "Auto-play audio failed. Please interact with the page to recover audio playback.",
    );
    ["click", "touch"].forEach((event) => {
        document.addEventListener(
            event,
            () => {
                element.setAttribute("muted", "false");
            },
            { once: true },
        );
    });
});
```

#### Detach Streaming

Use `detachStreaming` to stop rendering and release resources.

```javascript
const elements = await streaming.detachStreaming(channelId);
if (Array.isArray(elements)) {
    elements.forEach((element) => element.remove());
} else {
    elements.remove();
}
```

### 4. Event Listeners

Use these event listeners to monitor connection state changes and audio/video statistics.

#### Connection State Changes

Track the connection state with the streaming server:

```javascript
streaming.on("connection-change", (payload) => {
    const { state } = payload;
    if (state === "Connected") {
        console.log("Connected to the streaming service");
    } else if (state === "Fail") {
        console.log("Failed to connect to the streaming service");
    }
});
```

#### Audio/Video Statistics

Monitor statistics updates:

```javascript
streaming.on("audio-statistic-data-change", (payload) => {
  console.log("Audio Statistic Data Change:", payload);
});
streaming.on("video-statistic-data-change", (payload) => {
  console.log("Video Statistic Data Change:", payload);
});
Stream End Detection
element.addEventListener("ended", () => {
  console.log("Broadcast streaming ended");
});
```
