<!-- Source: https://developers.zoom.us/docs/video-sdk/web/video-best-practices.md -->
<!-- Fetched: 2026-06-15 -->

# Best practices


Here are some best practices to offer the best user experience for your integration.

## Rendering video

-   Implement [pagination](/docs/video-sdk/web/video/#video-pagination-example) or a swipe gesture to render the next set of user videos in the container. This is useful when you have more users than the amount of videos you can render (25 on desktop browsers, and 9 on mobile browsers). Pagination enables users to toggle through the videos to see them all.
-   Implement the names that are overlaid over the user video feeds.
-   Add `<meta charset="UTF-8" />` to your web app's entry point (`index.html`) to ensure the browser uses UTF-8. Without it, the browser might default to a different encoding, which can prevent users from accessing media features and trigger console errors immediately after a session is joined.

## Responsive design and aspect ratio

To provide a consistent user experience across different devices and window sizes, follow these responsive design best practices.

### Aspect ratio

By default, the Video SDK renders video at a **16:9** aspect ratio for desktop browsers and mobile devices in landscape mode. For mobile devices in portrait mode, it renders at **9:16**.

To ensure that the aspect ratio remains consistent when a user resizes their browser window or changes device orientation, use CSS to manage the video container. For example:

```css
video-player {
    width: 100%;
    height: auto;
    aspect-ratio: 16/9; /* or 9/16 for portrait */
}
```

If the sender's aspect ratio changes (for example, if they rotate their device), the [`video-aspect-ratio-change`](/docs/video-sdk/web/handle-events/#video-aspect-ratio-change) event triggers. You should listen for this event and adjust your UI accordingly.

### Video layout and density

While the Video SDK allows you to decide the size and number of videos to render, we recommend the following layouts for the best balance of performance and usability:

-   **Mobile**: Render 4 videos per page in a 2x2 grid.
-   **Desktop**: Render 9 videos per page in a 3x3 grid.

The Video SDK provides APIs to determine the maximum number of videos that can be rendered based on the device's capabilities (up to 25 on desktop and 9 on mobile). You can also use [pagination](/docs/video-sdk/web/video/#video-pagination-example) to show multiple videos.

### UI Toolkit

The [Zoom Video SDK UI Toolkit](/docs/video-sdk/web/ui-toolkit/) implements these responsive best practices and can be used as a reference or a recommended implementation for your own application.

## Cameras

-   Provide the option to choose a camera. This allows the end user to switch to their camera or choose their desired camera device. See [switch camera](#switch-camera) for details.

    ![Choose audio and video](/img/vsdk-web-bp5-chooseav.png)

-   Add support for hot plugging and unplugging. Use the `device-change` event listener to listen for when a camera is plugged in or unplugged. This can be useful to notify the user if their camera runs out of battery, loses connection, or for easily switching to a new device when one is connected or plugged in. See [`device-change`](/docs/video-sdk/web/sessions/#device-change) for details.
-   Show users' camera statuses so everyone can see that a user has their camera off or on.

    ![Show user list and audio video statuses](/img/uitk-web-UsersComponent2.png)

## Permission changes and errors

-   Handle system and browser permission errors, for example, if the end user has not granted camera access within the browser or webpage, or they have denied access, display the error and a call to action to the end user. You can also show a notification to the other users that this user is having trouble with allowing their camera to be enabled.
-   If session audio and video stop working due to a permission change, for example, perhaps the user disabled video permission in Chrome, you can use the[`permission-change`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.html#event_device_permission_change) event listener to detect this and message the user to authorize permission again.

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

## User changes

-   If the SDK is put into the background on a mobile device or if the device sleeps, the video turns off, but audio stays on. You don't need to do anything to turn the video back on when the SDK is made active again. If the user turns off the device or closes the app (such as by swiping up to close it in the App Switcher on iOS), the client side is destroyed. Use [`leaveOnPageUnload`](https://marketplacefront.zoom.us/sdk/custom/web/interfaces/ZoomVideo.InitOptions.html#leaveOnPageUnload) for the best experience when the user leaves the session in this way.
