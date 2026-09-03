<!-- Source: https://developers.zoom.us/docs/video-sdk/web/audio-advanced.md -->
<!-- Fetched: 2026-06-15 -->

# Advanced features


Learn how to control audio for each user with [granular audio control](#granular-audio-control), [share a second microphone](#share-a-second-microphone), and [play an audio media file](#play-audio-media-file).

## Granular audio control

In a single session, you can control the audio that each user hears with the `muteUserAudioLocally(userId)` function. This does not mute a user's microphone. Instead, it stops the SDK from playing a certain user's audio to the user this function is called for. For example, you can use this function to build a backstage experience. Or in a fitness class so that each user can hear the instructor, but not the other users taking the class.

In a similar manner, you can control the volume level you are hearing from other users with the `adjustUserAudioVolumeLocally(userId, volume)` function.

## Share second microphone audio

You can share audio from a second device, such as a stethoscope using the `startSecondaryAudio` function. This feature is available using the following platforms and browsers:

-   Chrome, Edge, and Firefox on Windows
-   Android browsers with audio compatible mode set to true

_**Safari and macOS not yet supported.**_

Once the user grants audio permission, get the list of microphones with `getMicList` and pass a `deviceId` into [`stream.startSecondaryAudio`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#startSecondaryAudio).

```javascript
let microphones = stream.getMicList();

stream.startSecondaryAudio(microphones[1].deviceId);
```

Use [`stream.stopSecondaryAudio`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#stopSecondaryAudio) to stop capturing audio from the second microphone.

## Play audio media file

You can use a media file to play audio in a session. Pass in the audio `url` as an object in the `mediaFile` property of the `startAudio` function.

```javascript
await mediaStream.startAudio({
    mediaFile: { url: "https://example.com/audio.mp3" },
});
```

> **Note:** If you want to use a video file (mp4) as both the video and the audio input, make sure the URL is the same for both the `startVideo` and `startAudio` functions. Be sure that the media file is from the same origin or has the necessary CORS headers for cross-origin access.

Use `switchMicrophone` to switch between a media file and a microphone device.

```javascript
await mediaStream.switchMicrophone({ url: "https://example.com/audio.mp3" });
```

To control media file playback, use `getAudioMediaPlaybackController` to obtain a controller. This allows you to use methods such as `play` and `pause` to control playback. You can also set the `playback` property to control whether the media file's sound is played locally.

```javascript
const controller = mediaStream.getAudioMediaPlaybackController();
// pause
controller.pause();
//play
controller.play();
// playback the sound
controller.playback = true;
```

See [Play video media file](/docs/video-sdk/web/video/#play-video-media-file) for instructions on how to use a media file to play video in a session.
