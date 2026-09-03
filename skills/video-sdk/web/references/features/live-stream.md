<!-- Source: https://developers.zoom.us/docs/video-sdk/web/live-stream.md -->
<!-- Fetched: 2026-06-15 -->

# Live stream (push)


Video SDK sessions can be live streamed to an unlimited audience using Real-Time Messaging Protocol (RTMP). This is helpful if you need to scale Video SDK attendees beyond the 1,000 real-time session user limit. For example, you can live stream to [YouTube Live](https://developers.google.com/youtube/v3/live/guides/rtmps-ingestion), [Facebook Live](https://www.facebook.com/help/755943624557739), [Twitch](<https://dev.twitch.tv/docs/video-broadcast/#:~:text=The%20tool%20sends%20a%20video,Time%20Messaging%20Protocol%20(RTMP).>), [AWS IVS](https://docs.aws.amazon.com/ivs/latest/LowLatencyUserGuide/streaming-config.html#streaming-config-settings), and other services.

See the following for how to set up a streaming event in a third-party platform and implement a feature to start and stop live streaming of a session using the Video SDK.

## Retrieve credentials

Live streaming with SDKs requires the following information from third-party streaming platforms: Stream URL, Stream Key, Broadcast URL. See [Live stream to YouTube](#live-stream-to-youtube) for an example.

To ingest an external RTMP stream into a session instead, see [Live stream (pull)](/docs/video-sdk/web/live-stream-pull/).

## Initialize the live stream client

After joining a session, call `client.getLiveStreamClient()` to get the live stream client.

```javascript
const liveStreamClient = client.getLiveStreamClient();
```

## Set and start the live stream

The Video SDK broadcasts the live stream to an RTMP endpoint. To receive the live stream, use an RTMP receiver like Youtube live, Facebook live, Twitch, or AWS IVS for an embedded, custom experience.

To set and start the live stream call the `liveStreamClient.startLiveStream()` function, passing in the `streamUrl` you are streaming to, the `key`, token, or live streaming URL password, and the `broadcastUrl`.

```javascript
liveStreamClient.startLiveStream(streamUrl, key, broadcastUrl);
```

The **broadcast URL** is the URL where an audience member can view the live stream. This may not be required from 3rd party live streaming platforms, but it is required by the `startLiveStream` function. This URL has no affect on the live stream, if you do not know what to pass in, you can pass in `https://zoom.us`.

## Stop the live stream

To stop the RTMP live stream, call:

```javascript
liveStreamClient.stopLiveStream();
```

## Live stream event listener

To receive the live streaming status, subscribe to the `live-stream-status` event. This event is useful for events like if the broadcaster stops the live stream on the receiving end.

```javascript
client.on(`live-stream-status`, (payload) => {
    console.log(`live streaming status: ${payload}`);
});
```

## Live stream to YouTube

For instance, if you want to live stream a session to YouTube, you must [enable live streaming](https://support.google.com/youtube/answer/9227509?hl=en) on your Google account.

### Step 1:

Login to YouTube. Locate the video icon and press "Go Live".

![Go Live option on YouTube](/img/1569946554166.png)

### Step 2:

Click the Stream button in the top panel. **Note: YouTube Webcam services are not compatible with Zoom SDK**.

![YouTube Stream button](/img/1569946563191.png)

### Step 3:

Fill out the required information and toggle "Schedule for later". If this is not selected, the live stream will start immediately and will not provide setting info.

![YouTube settings for Schedule for Later](/img/1569946580691.png)

### Step 4:

After creating the stream, the Steam URL and Stream Key will be available.

![YouTube Stream URL and Key setup](/img/1569946642209.png)

### Step 5:

Click the share button beside your account icon to get the broadcast URL.

![YouTube broadcast URL](/img/vsdk-ios-youtube-broadcastURL.png)

_Use the [YouTube Live Streaming API](https://developers.google.com/youtube/v3/live/getting-started) to automate these steps and get stream information programmatically._

## More live stream features

For the full set of live stream features, see [LiveStreamClient in the Video SDK Reference](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.LiveStreamClient.html).

## From the developer blog

See the following blogs for examples.

-   [How to live stream with the Zoom Video SDK](/blog/how-to-live-stream-with-video-sdk) by Will Ezrine - 03-07-2024
-   [On-demand playback scrubbing with Amazon IVS and Zoom Video SDK](/blog/video-playback-scrubbing-video-sdk) by James Coon - 03-06-2024
