<!-- Source: https://developers.zoom.us/docs/video-sdk/web/audio-sound-options.md -->
<!-- Fetched: 2026-06-15 -->

# Sound options


By default, Zoom uses noise suppression and echo cancellation to improve the quality of the audio received by your microphone, but these audio filters can interfere with situations that require capturing the full range of audio. See [Configuring professional audio settings for Zoom Meetings](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0059985) for details.

Video SDK uses the same technology to improve audio quality, but also supports the ability to turn off these filters to capture the **original sound** of the microphone, for example, to support the use of a high-quality microphone with built-in audio filters or to capture the full audio range without noise suppression. _**Capturing the microphone's original sound disables the background noise suppression feature.**_

Video SDK for web supports the following sound options.

-   Capturing original sound, including high fidelity and stereo audio
-   Background noise suppression

## Original sound

You can set original sound for `hifi` for high fidelity audio or `stereo` for stereo audio when connecting to session audio or after connecting. These options default to `false`. You can enable them together or separately.

### When connecting

Enable original sound when connecting to session audio using [`originalSound`](https://marketplacefront.zoom.us/sdk/custom/web/interfaces/ZoomVideo.AudioOption.html#originalSound).

```javascript
stream.startAudio({ originalSound: true });
```

Enable both the `hifi` and `stereo` options when connecting to session audio.

```javascript
stream.startAudio({
    originalSound: {
        stereo: true,
        hifi: true,
    },
});
```

### After connecting

Enable or disable these options after connecting to session audio.

To enable, set [`enableOriginalSound`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#enableOriginalSound) to `true`.

```javascript
stream.enableOriginalSound(true);
```

Enable both `hifi` and `stereo` options.

```javascript
stream.enableOriginalSound({
    hifi: true,
    stereo: true,
});
```

To disable, set [`enableOriginalSound`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#enableOriginalSound) to `false`.

```javascript
stream.enableOriginalSound(false);
```

Disable both `hifi` and `stereo` options.

```javascript
stream.enableOriginalSound({
    hifi: false,
    stereo: false,
});
```

## Background noise suppression

Enable advanced AI background noise suppression for all desktop browsers, which suppresses background noise in your environment like dogs barking, lawn mowers, clapping, fans, pen tapping, and other unwelcome distractions. For WebAssembly audio, this feature requires [`SharedArrayBuffer`](/docs/video-sdk/web/sharedarraybuffer/). It is not required for WebRTC audio.

You can enable it when connecting to session audio using [`backgroundNoiseSuppression`](https://marketplacefront.zoom.us/sdk/custom/web/interfaces/ZoomVideo.AudioOption.html#backgroundNoiseSuppression).

```javascript
stream.startAudio({ backgroundNoiseSuppression: true });
```

Or you can enable or disable it after connecting to session audio.

To enable, set [`enableBackgroundNoiseSuppression`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#enableBackgroundNoiseSuppression) to `true`.

```javascript
stream.enableBackgroundNoiseSuppression(true);
```

To disable, set [`enableBackgroundNoiseSuppression`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#enableBackgroundNoiseSuppression) to `false`.

```javascript
stream.enableBackgroundNoiseSuppression(false);
```
