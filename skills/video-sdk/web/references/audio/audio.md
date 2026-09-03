<!-- Source: https://developers.zoom.us/docs/video-sdk/web/audio.md -->
<!-- Fetched: 2026-06-15 -->

# Audio core features


Enable audio in your Video SDK app to allow users to communicate with each other. Users can connect using their computer's audio (microphone and speaker) or dial-in using [PSTN](/docs/video-sdk/web/pstn/) (phone).

This guide covers core audio features like starting, stopping, muting, and switching audio devices. See [audio best practices](/docs/video-sdk/web/audio-best-practices/) for suggestions on how to create the best user experience.

## Start audio

To connect to the session audio, call `stream.startAudio()`. This connects the user to the session audio, unmutes their microphone, and prompts the browser to ask for microphone permission the first time. When you do not specify any devices, the Video SDK uses the system default microphone and speaker. See [How the SDK selects audio devices](#how-the-sdk-selects-audio-devices) to choose specific devices.

### Call startAudio from a user gesture

Do not start audio automatically when the user joins a session. Call `stream.startAudio()` from an event handler triggered by a user action, like an "enable audio" button. If you call it outside of a user gesture, the browser may block microphone access. It may also fail to play audio, which dispatches the [`auto-play-audio-failed`](/docs/video-sdk/web/handle-events/#auto-play-audio-failed) event.

Modern versions of Chrome, Safari, and Firefox only require that the user has interacted with the page, such as through a click or touch, before audio capture begins. Even so, starting audio from a button click is still a best practice, as it reduces the risk of autoplay blocks and microphone permission failures.

The stricter requirement of calling `startAudio()` directly inside the gesture handler only applies to legacy versions of Safari (for example, Safari 15.2).

> **Note**
>
> Always call `stream.startAudio()` from a user-initiated event, like a button click. Otherwise, the browser might block programmatic access to the microphone or fail to play audio.

```javascript
// Tie startAudio to a real user action, such as a button click.
const enableAudioButton = document.querySelector("#enable-audio");

enableAudioButton.addEventListener("click", async () => {
    try {
        await stream.startAudio();
        console.log("Audio started");
    } catch (error) {
        // For example, the user denied microphone permission, or no device was found.
        console.error("Failed to start audio:", error);
        // Show the user a message and a way to try again.
    }
});
```

If the SDK fails to auto-play audio after `stream.startAudio()` is called, it dispatches the [`auto-play-audio-failed`](/docs/video-sdk/web/handle-events/#auto-play-audio-failed) event. Listen for it and prompt the user to interact with the page.

```javascript
client.on("auto-play-audio-failed", () => {
    // Prompt the user to click anywhere on the page to resume audio.
});
```

## How the SDK selects audio devices

The Video SDK always connects audio using a selected microphone and a selected speaker. You can let the SDK choose these devices, or specify them explicitly.

-   Default behavior - when you call `stream.startAudio()` without options, the SDK uses the system default microphone and speaker.
-   Explicit selection - to start audio with a specific device, pass a `microphoneId`, a `speakerId`, or both to `stream.startAudio()`.

```javascript
// Start audio with specific devices.
const microphones = stream.getMicList();
const speakers = stream.getSpeakerList();

await stream.startAudio({
    microphoneId: microphones[0]?.deviceId,
    speakerId: speakers[0]?.deviceId,
});
```

Switching the speaker is only supported in browsers that implement the [AudioContext API: setSinkId](https://caniuse.com/mdn-api_audiocontext_setsinkid). On unsupported browsers, like Safari, the SDK uses the system default speaker.

### Query the selected devices

After audio starts, query the selected devices and the full device lists at any time. Present the lists in your UI so users can choose a different device.

| Purpose                    | Microphone                          | Speaker                          |
| -------------------------- | ----------------------------------- | -------------------------------- |
| List available devices     | `stream.getMicList()`               | `stream.getSpeakerList()`        |
| Get the selected device    | `stream.getActiveMicrophone()`      | `stream.getActiveSpeaker()`      |
| Change the selected device | `stream.switchMicrophone(deviceId)` | `stream.switchSpeaker(deviceId)` |

Each device in a list is an object with a `deviceId` and a `label`. Pass a `deviceId` to the switch functions to change the selected device. See [Switch audio devices](#switch-audio-devices) for full examples.

### Handle device changes and fallback

If a selected device becomes unavailable, for example the user unplugs a headset, the system falls back to the default device and the SDK dispatches the [`device-change`](/docs/video-sdk/web/handle-events/#device-change) event. Listen for this event to refresh your device lists and update the selected device in your UI.

```javascript
client.on("device-change", () => {
    // Refresh the lists, then update your UI.
    const microphones = stream.getMicList();
    const speakers = stream.getSpeakerList();
    const activeMic = stream.getActiveMicrophone();
    const activeSpeaker = stream.getActiveSpeaker();
});
```

## Stop audio

To disconnect from the session audio, call `stream.stopAudio()`.

```javascript
stream.stopAudio();
```

> **Note**
>
> `stream.stopAudio()` completely disconnects the user from the audio portion of the session. Do not call this function when the user is not speaking; use `stream.muteAudio()` instead. `stream.stopAudio()` should only be used when the user has, for example, clicked a button to disconnect from the audio stream.

## Mute and unmute

To mute the user's own microphone, call `stream.muteAudio()`.

```javascript
stream.muteAudio();
```

To unmute their own microphone, call `stream.unmuteAudio()`.

```javascript
stream.unmuteAudio();
```

### Get audio status

To get a user's current audio status, get the user object and check the `audio` property.

For the current user:

```javascript
const currentUser = client.getCurrentUserInfo();
if (currentUser.muted === false) {
    console.log("User is unmuted");
} else {
    console.log("User is muted");
}
```

For any user in the session:

```javascript
const user = client.getUser(userId);
if (currentUser.muted === false) {
    console.log("User is unmuted");
} else {
    console.log("User is muted");
}
```

> **Note**
>
> `currentUser.muted` can be `true`, `false` or undefined (if audio isn't joined).

### Host can ask user to unmute

While only users can unmute themselves, a host can request a user to unmute by calling `stream.unmuteAudio()` with the user's `userId`.

```javascript
// Host calls this
stream.unmuteAudio(userId);
```

The target user's app receives the `host-ask-unmute-audio` event. You must handle this event and display a UI prompt to let the user accept or decline the request.

```javascript
// User receives this event
client.on("host-ask-unmute-audio", (payload) => {
    console.log(payload); // { userId: (ID of host who asked) }
    console.log("Host asked me to unmute");

    // Example: Show a confirmation dialog to the user
    if (window.confirm("The host has asked you to unmute. Unmute?")) {
        stream.unmuteAudio();
    }
});
```

## Detect active speaker

To identify who is currently speaking, use the `active-speaker` event. This event fires frequently, making it ideal for UI animations like highlighting the speaker's video frame or an audio icon.

```javascript
client.on("active-speaker", (payload) => {
    // payload is an array of active speakers
    // e.g., [{ userId: 123, displayName: 'Jane' }]
    console.log("Active speaker(s):", payload);
    // Update your UI to indicate who is speaking
});
```

For use cases where you want to change the video layout based on a more sustained period of speaking (e.g., pinning the speaker), use the `video-active-change` event. See [Video: Dynamically render active speaker](/docs/video-sdk/web/video/#dynamically-render-active-speaker) for details.

## Switch audio devices

You can get lists of available microphones and speakers and allow users to switch between them.

### Switch microphone

1. Get a list of available microphones using `stream.getMicList()`.
2. Get the currently active microphone using `stream.getActiveMicrophone()`.
3. To switch, pass the `deviceId` of the desired microphone to `stream.switchMicrophone()`.

We recommend presenting the list of microphones to the user in your UI.

```javascript
const microphones = stream.getMicList();
const activeMicrophoneId = stream.getActiveMicrophone();

// Example: Switch to the first available microphone
if (microphones.length) {
    stream.switchMicrophone(microphones[0].deviceId);
}
```

### Switch speaker

Similarly, you can switch the audio output device.

1. Get a list of available speakers using `stream.getSpeakerList()`.
2. Get the currently active speaker using `stream.getActiveSpeaker()`.
3. To switch, pass the `deviceId` of the desired speaker to `stream.switchSpeaker()`.

```javascript
const speakers = stream.getSpeakerList();
const activeSpeakerId = stream.getActiveSpeaker();

// Example: Switch to the first available speaker
if (speakers.length) {
    stream.switchSpeaker(speakers[0].deviceId);
}
```

## Troubleshooting audio

Use this table to map common audio issues to their likely causes and fixes.

| Issue                                                             | Likely cause                                                                                | Fix                                                                                                                                                                                                                  |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Audio does not start and `auto-play-audio-failed` fires           | `startAudio()` was called outside of a user gesture, or the browser blocked autoplay        | Call `startAudio()` from a user-initiated event. Listen for [`auto-play-audio-failed`](/docs/video-sdk/web/handle-events/#auto-play-audio-failed) and prompt the user to click the page.                             |
| `startAudio()` rejects with error `6010` (`AUDIO_CAPTURE_FAILED`) | Microphone permission was denied or dismissed, the device is in use, or no device was found | Check the reason and prompt the user to grant permission or choose another device. See [error codes](/docs/video-sdk/web/error-codes/#audio-exception).                                                              |
| Audio works in Chrome but not Safari                              | Safari does not persist permission and applies stricter autoplay rules                      | Request permission on each session and keep `startAudio()` inside the user gesture. Safari uses the system default speaker because it does not support `setSinkId`.                                                  |
| The selected speaker cannot be changed                            | The browser does not support the `setSinkId` API                                            | Hide the speaker picker when [`setSinkId`](https://caniuse.com/mdn-api_audiocontext_setsinkid) is unsupported. The SDK uses the system default speaker.                                                              |
| Audio drops after a phone call or another app uses the microphone | The OS or another app took over the microphone                                              | Listen for [`current-audio-change`](/docs/video-sdk/web/handle-events/#current-audio-change) and [`active-media-failed`](/docs/video-sdk/web/handle-events/#active-media-failed), then prompt the user to reconnect. |
| Switching a device has no effect                                  | The device list is stale                                                                    | Refresh the lists in the [`device-change`](/docs/video-sdk/web/handle-events/#device-change) handler before switching.                                                                                               |
| Audio stops working after the user changes a browser permission   | The user revoked microphone permission                                                      | Detect this with [`device-permission-change`](/docs/video-sdk/web/handle-events/#device-permission-change) and prompt the user to grant permission again.                                                            |

> **Note**
>
> Autoplay and permission behavior differ by browser. Chrome remembers a granted permission for the site, while Safari prompts on each session and requires the speaker to use the system default. Test your audio flow in both Chrome and Safari.

## More audio features

For the full set of audio features, see the [`Stream` class in the Video SDK Reference](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html). For the audio events referenced on this page, see [handle events](/docs/video-sdk/web/handle-events/).
