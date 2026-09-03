<!-- Source: https://developers.zoom.us/docs/video-sdk/web/audio-best-practices.md -->
<!-- Fetched: 2026-06-15 -->

# Best practices


Follow these best practices to give your users a smooth audio experience in your integration. For a quick reference, see the [Audio setup checklist](#audio-setup-checklist).

## Enable audio

-   Since browsers prevent audio from playing without user action, add an easy to identify "enable audio" button that your users can click, which calls the Video SDK `startAudio` function. Be sure to [tie this to a real user action](/docs/video-sdk/web/audio/#call-startaudio-from-a-user-gesture), like a button click, so audio initiates successfully within the browser.

    ![Enable audio button](/img/vsdk-web-bp2-audio.png)

## Audio initialization flow

The diagram below summarizes the correct order for starting audio, satisfying the browser gesture requirement, and handling devices through the session lifecycle.

```plaintext
User joins session
        |
        v
Show an "enable audio" button     (do NOT call startAudio() automatically)
        |
        v
User clicks the button            (this is the required user gesture)
        |
        v
Call stream.startAudio()
        |
        +--> Browser prompts for microphone permission (first time only)
        |
        v
Permission granted? --- no --> Listen for device-permission-change and
        |                       prompt the user to grant permission again
        | yes
        v
Audio connects with the selected microphone and speaker
        |
        v
Audio plays? --- no (auto-play-audio-failed) --> Prompt the user to click the page
        | yes
        v
In session: handle device-change, current-audio-change,
            and active-media-failed events for the rest of the lifecycle
```

## Show status

-   Animate the [microphone when users speak](/docs/video-sdk/web/audio/#detect-active-speaker). This helps users visualize that their mic is capturing their voice successfully.

    ![Microphone icon](/img/vsdk-web-bp3-mic.png)

-   Show users' microphone statuses so everyone can see that a user is muted, unmuted, or that they have not "enabled audio".

    ![Show user list and audio video statuses](/img/uitk-web-UsersComponent2.png)

-   [Highlight the active speaker](/docs/video-sdk/web/audio/#detect-active-speaker). If you have multiple users in the session, use the active speaker event to spotlight them in your UI, or border their video so everyone can visualize who is speaking.

## Audio device options and status

-   Provide the option to choose a [microphone](/docs/video-sdk/web/audio/#switch-microphone) and [speaker](/docs/video-sdk/web/audio/#switch-speaker) (if supported in the browser, see caniuse.com for the [AudioContext API: setSinkId](https://caniuse.com/mdn-api_audiocontext_setsinkid)). This allows the end user to switch to their headphones or choose their desired microphone device.

    ![Choose audio and video](/img/vsdk-web-bp5-chooseav.png)

-   Add support for hot plugging and unplugging. Use the `device-change` event listener to listen for when a microphone is plugged in or unplugged. This can be useful to notify the user if their headphones run out of battery, loose connection, or for easily switching to a new device when one is connected or plugged in. See [`device-change`](/docs/video-sdk/web/handle-events/#device-change) in **event handling** for details.
-   Use the [`current-audio-change`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.html#event_current_audio_change) event to display a message to ask the user to reconnect audio if they left audio for any reason, such as to take a phone call or play music. If they use another app for audio, it disconnects the SDK from audio and they need to reconnect. The user may not know this unless you tell them. See [`current-audio-change`](/docs/video-sdk/web/handle-events/#current-audio-change) in **event handling** for details.

## Permission changes and errors

-   Handle system and browser permission errors, for example, if the end user has not granted camera access within the browser or webpage, or they have denied access, display the error and a call to action to the end user. You can also show a notification to the other users that this user is having trouble with allowing their camera to be enabled.
-   If session audio and video stop working due to a permission change, for example, perhaps the user disabled audio permission in Chrome, you can use the[`permission-change`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.html#event_device_permission_change) event to detect this and message the user to authorize permission again.

### Support for Chrome `<permission>` element

Zoom supports the `<permission>` element in Chrome to give users contextual control over access permissions. Use it to enable the user to trigger the microphone and camera permission prompt in the browser by clicking it (it appears as a button).

```html
<style>
    permission {
        background-color: lightgray;
        color: black;
        border-radius: 10px;
    }
</style>
<permission type="geolocation"></permission>
```

This requires [the origin trial Trial for Page Embedded Permission Control - Cam/Mic/Geolocation](https://developer.chrome.com/origintrials/#/view_trial/4531184175088140289). See the following resources for details.

-   [Rethink web permissions: Seamless user control of powerful capabilities with Chrome's new proposed `<permission>` element](https://developer.chrome.com/blog/rethinking-web-permissions)
-   [The `<permission>` element - Seamless user control of powerful capabilities](https://github.com/WICG/PEPC/blob/main/explainer.md)

## Audio setup checklist

Use this checklist to confirm your audio integration follows the recommended flow.

-   Provide an "enable audio" button instead of starting audio automatically on join.
-   Call [`stream.startAudio()`](/docs/video-sdk/web/audio/#call-startaudio-from-a-user-gesture) only from a user gesture, like a click or tap.
-   Handle the [`auto-play-audio-failed`](/docs/video-sdk/web/handle-events/#auto-play-audio-failed) event and prompt the user to click the page.
-   Let the SDK use the [system default devices](/docs/video-sdk/web/audio/#how-the-sdk-selects-audio-devices), or pass a `microphoneId` and `speakerId` to `startAudio()`.
-   Present [microphone](/docs/video-sdk/web/audio/#switch-microphone) and [speaker](/docs/video-sdk/web/audio/#switch-speaker) lists so users can switch devices.
-   Refresh your device lists in the [`device-change`](/docs/video-sdk/web/handle-events/#device-change) handler.
-   Handle permission changes with [`device-permission-change`](/docs/video-sdk/web/handle-events/#device-permission-change).
-   Prompt users to reconnect when [`current-audio-change`](/docs/video-sdk/web/handle-events/#current-audio-change) or [`active-media-failed`](/docs/video-sdk/web/handle-events/#active-media-failed) fires.
-   Use [`muteAudio()` and `unmuteAudio()`](/docs/video-sdk/web/audio/#mute-and-unmute) for muting, and reserve `stopAudio()` for fully leaving audio.

### Related references

-   [Audio core features](/docs/video-sdk/web/audio/) - start, stop, mute, and switch audio devices.
-   [Handle events](/docs/video-sdk/web/handle-events/) - audio, device, and permission events.
-   [Error codes](/docs/video-sdk/web/error-codes/#audio-exception) - audio exceptions thrown by `startAudio`.
-   [`Stream` class reference](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html) - full audio method reference.
