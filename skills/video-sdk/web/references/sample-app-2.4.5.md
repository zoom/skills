# Web Sample Application 2.4.5

## Package identity

The reviewed `zoom-video-sdk-web-2.4.5.zip` archive contains:

- Application: `react-video-sdk-demo`
- Application version: `2.4.5`
- SDK dependency: `@zoom/videosdk` exactly `2.4.5`
- Framework baseline: React `18.3.1` with Vite
- Source: the Zoom Video SDK Web sample application

This archive is not the SDK library distribution. It contains no bundled `SKILL.md`, AI API-document pairs, or installed `@zoom/videosdk` declaration files. Use the sample as implementation evidence, then verify public contracts against the installed package's TypeScript declarations and official API reference.

## Source map

| Area | Sample source |
|------|---------------|
| Session bootstrap | `src/index.tsx`, `src/context/` |
| Video rendering and pagination | `src/feature/video/video-attach.tsx`, `src/feature/video/hooks/useAttachPagination.ts` |
| RTMS controls | `src/feature/video/components/real-time-media-streams.tsx`, `video-footer.tsx` |
| Incoming RTMP | `src/feature/video/components/rtmp-ingestion-modal.tsx` |
| Broadcast host controls | `src/feature/video/components/live-stream.tsx` |
| Broadcast viewer | `src/streaming-viewer/` |
| Whiteboard | `src/feature/video/components/whiteboard.tsx`, `whiteboard-view.tsx`, `hooks/useWhiteboard.ts` |
| Voice translation | `src/feature/video/components/voice-translator.tsx`, `hooks/useVoiceTranslator.ts` |
| Audio/video processors | `src/processor/`, `src/feature/video/components/video-footer.tsx` |
| Video masks | `src/feature/video/components/video-mask-modal.tsx` |
| Chat and files | `src/feature/chat/hooks/useChat.ts` |
| Subsessions | `src/feature/subsession/` |

## RTMS lifecycle

The Web SDK controls whether the session publishes RTMS:

```typescript
const rtms = client.getRealTimeMediaStreamsClient();

if (rtms.isSupportRealTimeMediaStreams()) {
  await rtms.startRealTimeMediaStreams();
}

client.on('real-time-media-streams-status-change', (status) => {
  // Drive UI from RealTimeMediaStreamsStatus.
});

await rtms.pauseRealTimeMediaStreams();
await rtms.resumeRealTimeMediaStreams();
await rtms.stopRealTimeMediaStreams();
```

Treat the status event as the state authority rather than assuming a resolved control call means the transition completed. For RTMS backend signaling, media sockets, protocol messages, and processing, use [zoom-rtms](../../../rtms/SKILL.md).

## Incoming RTMP

The sample combines backend stream-ingestion management with in-session binding:

```typescript
const incoming = client.getIncomingLiveStreamClient();

await incoming.bindIncomingLiveStream(streamId);
await incoming.startIncomingLiveStream(streamId);

client.on('incoming-live-stream-status', (status) => {
  // status.isRTMPConnected, status.isStreamPushed, status.streamId
});

await incoming.stopIncomingLiveStream(streamId);
await incoming.unbindIncomingLiveStream(streamId);
```

The sample casts this client through `any`, and its create/list/delete endpoints are placeholders. Verify `getIncomingLiveStreamClient()` against the installed 2.4.5 declarations and implement authenticated backend endpoints before using this flow.

## Broadcast streaming

Host-side control uses `client.getBroadcastStreamingClient()` and the `broadcast-streaming-status` event. Viewer-side rendering uses the separate entry point:

```typescript
import ZoomStreaming, { VideoQuality } from '@zoom/videosdk/broadcast-streaming';

const streaming = ZoomStreaming.createClient({
  dependentAssets: `${window.location.origin}/lib`
});

await streaming.attachStreaming(channelId, signature, VideoQuality.Video_720P, liveVideoElement);
await streaming.detachStreaming(channelId, liveVideoElement);
```

Handle `connection-change`, `auto-play-audio-failed`, and the `<live-video>` element's `ended` event.

## Whiteboard

Whiteboard APIs require explicit presentation/view containers:

```typescript
const whiteboard = client.getWhiteboardClient();

await whiteboard.startWhiteboardScreen(container);
await whiteboard.startWhiteboardView(container, presenterUserId);
await whiteboard.exportWhiteboard('pdf');
await whiteboard.stopWhiteboardView();
await whiteboard.stopWhiteboardScreen();
```

Listen for `whiteboard-status-change`, `peer-whiteboard-state-change`, and `passively-stop-whiteboard`. Export before stopping when the content must be retained.

## Processors and masks

Feature-gate processors before presenting controls:

```typescript
if (stream.isSupportVideoProcessor()) {
  const processor = await stream.createProcessor({
    url: '/static/processors/watermark-processor.js',
    type: 'video',
    name: 'watermark-processor',
    options: {}
  });

  await stream.addProcessor(processor);
  await stream.removeProcessor(processor);
}
```

The sample includes bypass, white-noise, pitch-shift, and watermark processors. It also demonstrates `previewVideoMask`, `updateVideoMask`, and `stopPreviewVideoMask`.

## Security and compatibility guardrails

- Never ship `sdkSecret` or the sample's browser-side JWT generator. Issue Video SDK JWTs from a backend.
- Do not treat placeholder RTMP backend URLs or use of the session signature as an application authorization design.
- Register matching `on`/`off` handlers and clean up media, views, processors, and object URLs.
- Feature-gate optional APIs because account entitlements, browser support, and SDK version affect availability.
- React `18.3.1` is only the sample's tested baseline; it is not evidence for or against React 19 support.
- Treat sample source as an implementation pattern, not a replacement for public TypeScript declarations.
