<!-- Source: https://developers.zoom.us/docs/video-sdk/web/video-hd.md -->
<!-- Fetched: 2026-06-15 -->

# HD video


Video SDK for web supports sending up to 1080p that native clients can render and rendering up to 1080p video streams. You can render one 1080p video on a webpage at a time. The SDK automatically scales the video resolution depending on device capabilities and network conditions.

-   For designs like speaker view, where the active speaker video is larger than the other users, we recommend that you render the active speaker's video in 720p or 1080p and other smaller user videos in 180p.
-   For designs like gallery view, where all user videos are the same size, we recommend that you render the videos in 360p. As a best practice:
    -   Gallery View 3x3, medium sized videos in 360p.
    -   Gallery View 5x5, smaller sized videos in 180p.
-   For a view where the videos are small, we recommend rendering them as 180p. For example, as small avatars, or when you are rendering screen share content that takes up most of the webpage, while also rendering small user videos alongside the screen share content.

_**See [the blog](#from-the-developer-blog) for more.**_

## Determine HD video support

Use [`isSupportHDVideo`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#isSupportHDVideo) to check if the account is capable of 720p video. If the account or if CPU load and network conditions are not capable, 720p and 1080p video may not be able to be sent.

```javascript
stream.isSupportHDVideo();
```

## Determine maximum video number

If you want to render as many 720P videos as possible, you can use [`getMaxRenderableVideos`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#getMaxRenderableVideos) to determine the the maximum number of 720P videos that you can render.

```typescript
switch (stream.getMaxRenderableVideos()) {
    case 1:
        // You can render a 720P video.
        break;
    case 4:
        // You can render a 720P video and three 360P videos.
        break;
    case 9:
        // You can render two 720P videos and seven 180P videos.
        break;
    case 25:
        // You can render two 720P videos and twenty three 90P videos.
        break;
}
```

You can also use this method to automatically determine the layout.

```javascript
if (stream.getMaxRenderableVideos() > 1) {
    // gallery view layout
} else {
    // active speaker view layout (render the self view and a remote user video like the active speaker)
}
```

## Send 720p video

To capture and send 720p video from the camera, pass the `hd:true` property in [`startVideo`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#startVideo).

```javascript
stream.startVideo({ hd: true });
```

## Send 1080p video

To capture and send 1080p video from the camera, pass the `fullHd:true` property in [`startVideo`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#startVideo).

```javascript
stream.startVideo({ fullHd: true });
```

## Render 720p or 1080p video

To receive and render 720p video, pass `Video_720P` to the `videoQuality` parameter in [`attachVideo`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#attachVideo).

```javascript
stream.attachVideo(userId,VideoQuality.Video_720P, element?: string | VideoPlayer)
```

To receive and render 1080p video, pass `Video_1080P` to the `videoQuality` parameter in [`attachVideo`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#attachVideo).

```javascript
stream.attachVideo(userId,VideoQuality.Video_1080P, element?: string | VideoPlayer)
```

## Verify 720p or 1080p video

To verify the video resolution the Video SDK is receiving, use the `video-statistic-data-change` event listener. First, call [`subscribeVideoStatisticData`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#subscribeVideoStatisticData) to initialize the event listener. Ensure that the camera is turned on.

```javascript
stream.subscribeVideoStatisticData();

client.on("video-statistic-data-change", (payload) => {
    console.log(payload.height); // Outputs 360, 720, or 1080
});
```

## From the developer blog

See the following blog post for an example of video resolution.

-   [Video Resolution with the Video SDK](/blog/video-resolution-with-the-video-sdk) by Rehema Armorer - 02-09-2024
