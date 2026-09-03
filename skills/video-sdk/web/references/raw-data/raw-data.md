<!-- Source: https://developers.zoom.us/docs/video-sdk/web/raw-data.md -->
<!-- Fetched: 2026-06-15 -->

# Raw data


The Video SDK gives you access to real-time raw audio, video, and share data during a session, so you can apply additional effects and enhance the user experience. Use a processor component to apply effects or transformations to each piece of data in real-time.

-   Use [video](/docs/video-sdk/web/raw-data-video/) processors to modify each video frame in real-time.
-   Use [audio](/docs/video-sdk/web/raw-data-audio/) processors to modify audio data in real-time.
-   Use [share](/docs/video-sdk/web/raw-data-share/) processors to modify shared screen data in real-time.

## Limitations

-   The SDK allows only one active processor of the same type (video, audio, or share) at any given time.

> **Note**
>
> Each processor must be created only once during a session. Attempting to create multiple instances of the same processor will result in an error. If the user rejoins the session or a failover occurs, you may create a new instance of the processor as needed.

## Steps to implement a processor

1. Check if the browser supports custom processors. Use:
    - `isSupportVideoProcessor` for video processors
    - `isSupportAudioProcessor` for audio processors
    - `isSupportShareProcessor` for share processors
2. Build a custom processor.
3. Create a processor instance and add it to the video, audio, or share pipeline.

These are inspired by [`AudioWorklet`](https://developer.mozilla.org/en-US/docs/Web/API/AudioWorklet) processors.

## Samples

See the samples for examples of simple implementations.

-   [Zoom Media Processor Sample](https://github.com/zoom/videosdk-web-processor-sample)
