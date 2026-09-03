<!-- Source: https://developers.zoom.us/docs/video-sdk/web/video-advanced.md -->
<!-- Fetched: 2026-06-15 -->

# Advanced features


Learn how to [add virtual backgrounds](#add-virtual-background), [play video media files](#play-video-media-file), and [set rounded corners for video elements](#set-rounded-corners-for-video).

## Add virtual background

You can enable virtual background using WebAssembly video with [`SharedArrayBuffer`](/docs/video-sdk/web/sharedarraybuffer/) (SAB) enabled for better performance or without SAB and the [`enforceVirtualBackground`](https://marketplacefront.zoom.us/sdk/custom/web/interfaces/ZoomVideo.InitOptions.html#enforceVirtualBackground) `init` option to `true`. Or using WebRTC video. In either case, use SAB for better performance.

On desktop browsers, to set a virtual background or blurred background, pass in an image URL hosted from your domain, or the string `blur`, to the [`startVideo`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#startVideo) function.

```javascript
stream.startVideo({ virtualBackground: { imageUrl: "./my-background.jpg" } });
```

## Play video media file

You can use a media file to play video in a session. Pass in the video `url` as an object in the `mediaFile` property of the `startVideo` function.

```javascript
await mediaStream.startVideo({
    mediaFile: { url: "https://example.com/video.mp4" },
});
```

> **Note:** If you use the same file for both the video and the audio input, when you stop the video the audio output will also be paused. Be sure that the media file is from the same origin or has the necessary CORS headers for cross-origin access.

Use `switchCamera` to switch between a media file and a camera device.

```javascript
await mediaStream.switchCamera({ url: "https://example.com/video.mp4" });
```

To control media file playback, use `getVideoMediaPlaybackController` to obtain a controller. This allows you to use methods such as `play` and `pause` to control playback.

```javascript
const controller = mediaStream.getVideoMediaPlaybackController();
// pause
controller.pause();
//play
controller.play();
```

See [Play audio media file](/docs/video-sdk/web/audio/#play-audio-media-file) for instructions on how to use a media file to play audio in a session.

## Set rounded corners for video

The `video-player` tags support CSS styling, for example, use the CSS `border-radius` property to set rounded corners.

```css
video-player {
    border-radius: 20px;
}
```

See the **mdn web docs** for [`border-radius`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-radius#syntax) for details.
