<!-- Source: https://developers.zoom.us/docs/video-sdk/web/live-stream-pull.md -->
<!-- Fetched: 2026-07-17 -->

# Live stream (pull)


The incoming live stream feature lets you ingest an external RTMP stream (from OBS, vMix, hardware encoders, or other software) into a Video SDK session as a special virtual participant. Other session participants can subscribe to its video and audio just like any other participant.

> [**Contact Sales**](https://explore.zoom.us/en/video-sdk/#sf_form) to enable.

To broadcast a session out to an RTMP endpoint instead, see [Live stream (push)](/docs/video-sdk/web/live-stream/).

## Constraints

-   Only the session **host** can bind, start, stop, or unbind a stream.
-   Only **one** incoming stream is supported per session at a time.
-   Cannot be used from within a breakout room.

## Common use cases

-   Broadcasting pre-recorded content into a live session
-   Integrating external streaming software (OBS, vMix) as a session participant
-   Creating hybrid events with both live participants and streamed content

## Prerequisites

-   A Zoom Video SDK account with the incoming live stream feature enabled
-   Host privileges in the active session
-   RTMP streaming software (OBS, vMix, or a hardware encoder) available for configuration

## Get the incoming live stream client

After joining a session, retrieve the feature client:

```javascript
const incomingLiveStream = client.getIncomingLiveStreamClient();
```

## Step 1: Create a stream ingestion via the REST API

Before calling any SDK method, use the Zoom REST API to create a stream ingestion. This returns the credentials your streaming software needs.

```bash
POST https://api.zoom.us/v2/videosdk/stream_ingestions
```

### Request body

```json
{
    "session_id": "your_session_id",
    "stream_name": "My Incoming Stream"
}
```

### Response

```json
{
    "stream_id": "abc123def456",
    "stream_url": "rtmp://stream.zoom.us/live",
    "stream_key": "sk_your_stream_key"
}
```

| Field        | Purpose                                                     |
| ------------ | ----------------------------------------------------------- |
| `stream_id`  | Pass as `streamId` to all SDK methods                       |
| `stream_url` | Configure as the RTMP server URL in your streaming software |
| `stream_key` | Configure as the stream key in your streaming software      |

See [Create a stream ingestion](/docs/api/video-sdk/#tag/sessions/post/videosdk/stream_ingestions) in the API reference.

## Step 2: Bind the stream

Call `bindIncomingLiveStream` with the `stream_id` from the REST API response to associate the stream with the current session:

```javascript
const streamId = "abc123def456"; // stream_id from REST API response

await incomingLiveStream.bindIncomingLiveStream(streamId);
```

> **Note**
>
> If another stream is already RTMP-connected, binding a different stream ID will fail. Unbind the existing stream first.

## Step 3: Configure and start your streaming software

After binding, configure your streaming software (e.g., OBS Studio) with the credentials from the REST API response:

1. Open OBS → Settings → Stream
2. Set **Server** to the `stream_url` value
3. Set **Stream Key** to the `stream_key` value
4. Click **Start Streaming**

The stream must be actively pushing video before you can call `startIncomingLiveStream`.

## Step 4: Check stream status

Use `getIncomingLiveStreamStatus()` to verify the stream is ready. When a stream is bound but not yet RTMP-connected, each call also triggers an on-demand status refresh from the server.

```javascript
const status = incomingLiveStream.getIncomingLiveStreamStatus();
// { isRTMPConnected: boolean, isStreamPushed: boolean, streamId: string }

if (status.isRTMPConnected && !status.isStreamPushed) {
    // Streaming software is connected, ready to call startIncomingLiveStream
}
```

| `isRTMPConnected` | `isStreamPushed` | Meaning                                                                                                |
| ----------------- | ---------------- | ------------------------------------------------------------------------------------------------------ |
| `false`           | `false`          | Streaming software not yet connected. Check your RTMP URL and stream key                               |
| `true`            | `false`          | Streaming software is connected and pushing, but the host has not yet called `startIncomingLiveStream` |
| `true`            | `true`           | Stream is active as a virtual participant in the session                                               |

For real-time updates instead of polling, listen to the [`incoming-live-stream-status`](#handle-status-events) event.

## Step 5: Start the stream as a virtual participant

Once `isRTMPConnected` is `true`, call `startIncomingLiveStream`. The stream joins the session as a virtual participant named **"Incoming livestream"**.

```javascript
await incomingLiveStream.startIncomingLiveStream(streamId);
// Resolves when isStreamPushed becomes true
```

The promise resolves only after the server confirms the stream is actively pushing (`isStreamPushed: true`).

## Step 6: Subscribe to the stream

Once started, the stream appears in the participant list like any other user. Identify it using the `user-added` event and the participant's `isRtmpUser` flag:

```javascript
client.on("user-added", (participants) => {
    const streamUser = participants.find((p) => p.isRtmpUser);
    if (streamUser) {
        client
            .getMediaStream()
            .attachVideo(streamUser.userId, VideoQuality.Video_720P)
            .then((userVideo) => {
                document
                    .querySelector("video-player-container")
                    .appendChild(userVideo);
            });
    }
});
```

## Step 7: Stop and unbind

Stopping and unbinding are two separate steps with a required prerequisite:

1. **Stop** - removes the stream as a virtual participant from the session (if active).
2. **Stop pushing in your streaming software** - `unbindIncomingLiveStream` requires `isRTMPConnected` to be `false`. Listen to the `incoming-live-stream-status` event and wait until the streaming software has disconnected before calling unbind.
3. **Unbind** - releases server-side resources once the RTMP connection is fully released.

```javascript
// Step 1: stop the virtual participant if stream is active
const { isStreamPushed } = incomingLiveStream.getIncomingLiveStreamStatus();
if (isStreamPushed) {
    await incomingLiveStream.stopIncomingLiveStream(streamId);
}

// Step 2: stop pushing in your streaming software (OBS, vMix, etc.)
// Then listen for the RTMP connection to drop before unbinding

// Step 3: unbind once isRTMPConnected becomes false
client.on("incoming-live-stream-status", async (payload) => {
    if (!payload.isRTMPConnected) {
        await incomingLiveStream.unbindIncomingLiveStream(payload.streamId);
    }
});
```

> **Note**
>
> Calling `unbindIncomingLiveStream` while `isRTMPConnected` is still `true` will reject with an error. Always ensure the streaming software has stopped pushing first.

## Step 8: Delete the stream ingestion via the REST API

After unbinding, delete the stream ingestion from your **server**. Each Video SDK account has a limit on the number of stream ingestions, so cleaning up unused ones prevents hitting that ceiling.
Call the following endpoint from your backend after `unbindIncomingLiveStream` resolves:

```bash
DELETE https://api.zoom.us/v2/videosdk/stream_ingestions/{streamId}
```

A successful deletion returns `204 No Content`.

> **Important**
>
> The stream ingestion must be fully unbound before deletion. Attempting to delete an in-use ingestion returns error `34015`. Only call this endpoint after the SDK's `unbindIncomingLiveStream` promise has resolved.

| Error Code | Meaning                                           |
| ---------- | ------------------------------------------------- |
| `34015`    | Stream ingestion is still in use; unbind it first |
| `34012`    | Failed to delete the stream ingestion             |
| `34001`    | Stream ingestion does not exist                   |

## Handle status events

Listen to `incoming-live-stream-status` for real-time status changes during the session, useful for driving start/stop logic reactively instead of polling:

```javascript
client.on("incoming-live-stream-status", (payload) => {
    const { isRTMPConnected, isStreamPushed, streamId } = payload;

    if (isRTMPConnected && !isStreamPushed) {
        // Streaming software is connected and pushing, start as a virtual participant
        incomingLiveStream.startIncomingLiveStream(streamId);
    } else if (!isRTMPConnected) {
        // Streaming software has stopped pushing, safe to unbind
        incomingLiveStream.unbindIncomingLiveStream(streamId);
    }
});
```

## Complete integration example

```javascript
const streamId = "abc123def456"; // stream_id from POST /v2/videosdk/stream_ingestions

// Retrieve client after joining the session
const incomingLiveStream = client.getIncomingLiveStreamClient();

// 1. Bind the stream
await incomingLiveStream.bindIncomingLiveStream(streamId);

// 2. Configure OBS/vMix with stream_url + stream_key, then start streaming

// 3. React to status changes
client.on("incoming-live-stream-status", async (payload) => {
    const { isRTMPConnected, isStreamPushed, streamId } = payload;

    if (isRTMPConnected && !isStreamPushed) {
        await incomingLiveStream.startIncomingLiveStream(streamId);
        console.log("Incoming stream is now a virtual participant");
    }
});

// 4. Subscribe to the stream's video when the participant joins
client.on("user-added", (participants) => {
    const streamUser = participants.find((p) => p.isRtmpUser);
    if (streamUser) {
        client
            .getMediaStream()
            .attachVideo(streamUser.userId, VideoQuality.Video_720P)
            .then((userVideo) => {
                document
                    .querySelector("video-player-container")
                    .appendChild(userVideo);
            });
    }
});

// 5. On session end, stop the virtual participant if active
// unbind is triggered via incoming-live-stream-status once isRTMPConnected becomes false
async function cleanup() {
    const { isStreamPushed } = incomingLiveStream.getIncomingLiveStreamStatus();
    if (isStreamPushed) {
        await incomingLiveStream.stopIncomingLiveStream(streamId);
    }
    // Ask the user to stop pushing in their streaming software.
    // unbindIncomingLiveStream will be called by the incoming-live-stream-status handler above
    // once isRTMPConnected becomes false.
}
```
