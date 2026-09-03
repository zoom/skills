<!-- Source: https://developers.zoom.us/docs/video-sdk/web/get-started.md -->
<!-- Fetched: 2026-06-15 -->

# Get started


> **Action Required**
>
> If you use WebRTC video, we recommend that you upgrade to 2.3.15 or above to prevent issues with future Chrome versions.

Use the [Zoom Video SDK UI Toolkit](/docs/video-sdk/web/ui-toolkit/), a prebuilt user interface, to include the Video SDK in your application or integrate Video SDK directly to customize the experience yourself.

> ## Quickstart
>
> If you're building with React, use our [React Library for Zoom Video SDK](https://github.com/zoom/videosdk-react/) to get started faster with ready-to-use hooks and components.

## Prebuilt user interface

The [Zoom Video SDK UI Toolkit](/docs/video-sdk/web/ui-toolkit/) is a prebuilt user interface that you can use to quickly add Video SDK to your app.

1. [Get your credentials](/docs/video-sdk/get-credentials/#get-video-sdk-credentials).
2. [Install the UI Toolkit in your app](/docs/video-sdk/web/ui-toolkit/#install).

See the documentation for details on how to start and join sessions and use the prebuilt Video SDK features. The UI Toolkit is all you need to run the Video SDK in your app.

## Custom user interface

For a more sophisticated integration, you can integrate the SDK and features directly into your app. To get started, install the Video SDK for web via npm or the Video SDK CDN.

To install the [Video SDK via npm](https://www.npmjs.com/package/@zoom/videosdk), from the terminal in your project directory, run:

```shell
npm install @zoom/videosdk --save
```

After you've installed from npm, import `ZoomVideo` in the component file where you want to use the Video SDK:

```javascript
import ZoomVideo from "@zoom/videosdk";
```

To install the Video SDK via CDN, add the following script to the HTML page that includes the Video SDK, or to the `index.html` file if you are using a single-page application (SPA) framework.

```html
<script src="https://source.zoom.us/videosdk/zoom-video-2.2.12.min.js"></script>
```

Replace `2.2.12` with the [latest Video SDK for Web version number](https://www.npmjs.com/package/@zoom/videosdk).

If the CDN returns a `forbidden` error, verify that the URL is correct.

After installing the SDK from the CDN, you can access the `ZoomVideo` object from the global `window.WebVideoSDK` in the component where you want to use the SDK.

```javascript
const ZoomVideo = window.WebVideoSDK.default;
```

## Use UTF-8 encoding

Add `<meta charset="UTF-8" />` to your web app's entry point (`index.html`) to ensure the browser uses UTF-8. Without it, the browser might default to a different encoding, which can prevent users from accessing media features and trigger console errors immediately after a session is joined.

### Start and join sessions

A Video SDK session connects two or more users for realtime, two-way communication. Zoom creates sessions on demand; they don't need to be scheduled. You can run an unlimited amount of sessions concurrently. Up to 5,000 users can join a session. A user can be in one session per browser tab at a time.

To start and join sessions, after you've installed the Video SDK, create the client, `init` the Video SDK, and declare a variable called `stream`. Then, call the `join` function and pass in a session name, [Video SDK JWT](/docs/video-sdk/auth/#generate-a-video-sdk-jwt), `username`, and `sessionPasscode`. After joining the session, define `stream` to `client.getMediaStream()`. You'll use this namespace to build out features.

```javascript
import ZoomVideo from "@zoom/videosdk";

var client = ZoomVideo.createClient();
var stream;

client.init("en-US", "Global", { patchJsMedia: true }).then(() => {
    client
        .join("sessionName", "VIDEO_SDK_JWT", "userName", "sessionPasscode")
        .then(() => {
            stream = client.getMediaStream();
        });
});
```

```javascript
import ZoomVideo from "@zoom/videosdk";

var client = ZoomVideo.createClient();
var stream;

await client.init("en-US", "Global", { patchJsMedia: true });
await client.join(
    "sessionName",
    "VIDEO_SDK_JWT",
    "userName",
    "sessionPasscode",
);

stream = client.getMediaStream();
```

-   The session begins when the first user joins.
-   The session name must match the `tpc` in the Video SDK JWT.
-   The host of the session is the user with a `role` set to `1` in the [Video SDK JWT](/docs/video-sdk/auth/).
-   The `sessionPasscode` is optional, but if set for a session, it's required for other users joining that session.

See [Sessions](/docs/video-sdk/web/sessions/) for details about the session lifecycle, including best practices for pre-session, in-session, and post-session, how to leave and end sessions, and managing user roles and actions. See [Handle events](/docs/video-sdk/web/events) for how to best use event listeners.

See the sample for an example of a simple implementation.

-   [Video SDK for web sample](https://github.com/zoom/videosdk-web-sample)

### Add features

Add features such as [video](/docs/video-sdk/web/video/), [audio](/docs/video-sdk/web/audio/), [screen sharing](/docs/video-sdk/web/share/), [chat](/docs/video-sdk/web/chat/), and more.

---

_Use of this SDK is subject to our [Video SDK Terms of Service](https://explore.zoom.us/en/video-sdk-terms). See [Video SDK Plans & Pricing for Developer](https://zoom.us/pricing/developer) for pricing._
