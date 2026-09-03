<!-- Source: https://developers.zoom.us/docs/video-sdk/web/browser-support.md -->
<!-- Fetched: 2026-07-17 -->

# Browser support


The Zoom Video SDK for web [supports browsers](#check-device-or-browser-support) on this page within two (2) versions of their current version. The following table shows the features supported by browser. See the numbered feature support [footnotes](#footnotes) and [Mobile and tablet browser support](#mobile-and-tablet-browser-support) for details.

| Feature                                        | Chrome               | Firefox              | Safari               | Edge                 | [iOS/iPadOS](#mobile-and-tablet-browser-support) | [Android](#mobile-and-tablet-browser-support) |
| ---------------------------------------------- | -------------------- | -------------------- | -------------------- | -------------------- | ------------------------------------------------ | --------------------------------------------- |
| 1080p video (receive)                          | ✔                   | ✘                    | ✘                    | ✔                   | ✘                                                | ✘                                             |
| 1080p video (send) [(1)](#footnotes)           | ✔                   | ✘                    | ✘                    | ✔                   | ✘                                                | ✘                                             |
| 720p video (receive)                           | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| 720p video (send) [(2)](#footnotes)            | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Audio (receive)                                | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Audio (send)                                   | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Background noise suppression [(3)](#footnotes) | ✔                   | ✔                   | ✔                   | ✔                   | ✘                                                | ✘                                             |
| Call out (PSTN)                                | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Chat                                           | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Chat - send file                               | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Closed captioning                              | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Cloud recording                                | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Command channel                                | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Encryption [(4)](#footnotes)                   | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| End-to-end encryption (E2EE) [(5)](#footnotes) | ✘                    | ✘                    | ✘                    | ✘                    | ✘                                                | ✘                                             |
| Live transcription                             | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Live translation                               | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Render multiple videos                         | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| RTMP live streaming                            | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Screen share (receive)                         | ✔                   | ✔                   | ✔                   | ✔ [(6)](#footnotes) | ✔                                               | ✔                                            |
| Screen share (send)                            | ✔                   | ✔                   | ✔                   | ✔                   | ✘                                                | ✘                                             |
| Share second camera                            | ✔                   | ✔                   | ✔                   | ✔                   | ✘                                                | ✘                                             |
| Share tab audio                                | ✔                   | ✘                    | ✘                    | ✔                   | ✘                                                | ✘                                             |
| Video (receive)                                | ✔                   | ✔                   | ✔                   | ✔ [(6)](#footnotes) | ✔                                               | ✔                                            |
| Video (send)                                   | ✔                   | ✔                   | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Virtual background                             | ✔ [(7)](#footnotes) | ✔ [(7)](#footnotes) | ✔ [(7)](#footnotes) | ✔                   | ✘                                                | ✘                                             |
| WebRTC video [(8)](#footnotes)                 | ✔                   | ✘                    | ✔                   | ✔                   | ✔                                               | ✔                                            |
| Share system audio [(9)](#footnotes)           | ✔                   | ✘                    | ✘                    | ✔                   | ✘                                                | ✘                                             |

## Footnotes

1. [1080p video (send)](/docs/video-sdk/web/video/#send-1080p-video) requires WebAssembly and Chromium browser and machines with at least 8 cores, a GPU or hardware acceleration, [`SharedArrayBuffer`](/docs/video-sdk/web/sharedarraybuffer/), and quality network connections. It is not supported by WebRTC video. See [Navigator: hardwareConcurrency property](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/hardwareConcurrency), [webcodecs support](https://caniuse.com/webcodecs), and [HD Video](/docs/video-sdk/web/video/#hd-video) for details.
2. [720p video (send)](/docs/video-sdk/web/video/#send-720p-video) requires [`SharedArrayBuffer`](/docs/video-sdk/web/sharedarraybuffer/) for WebAssembly video, but not for WebRTC video. It's supported in landscape mode on mobile browsers on iOS >= 16.4 and Android Chrome >= 112.
3. Background noise suppression requires [`SharedArrayBuffer`](/docs/video-sdk/web/sharedarraybuffer/) for WebAssembly audio, but not for WebRTC audio.
4. The Zoom Video SDK uses TLS 1.2 and 256-bit AES-GCM Encryption. See the [Zoom Encryption whitepaper](https://explore.zoom.us/docs/doc/Zoom%20Encryption%20Whitepaper.pdf) for details.
5. [End-to-end encryption (E2EE)](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0065408) is **not** supported on any browser.
6. Known issue: On desktop browsers using Microsoft Edge, users may see a purple background around text when receiving video. This is due to the [Microsoft 'Enhance your security on the web' feature, in strict mode](https://support.microsoft.com/en-us/microsoft-edge/enhance-your-security-on-the-web-with-microsoft-edge-b8199f13-b21b-4a08-a806-daed31a1929d). The user can switch this feature off or change to Balanced mode, or you can add your URL needs to an exception list to properly display the video. See [the link](https://support.microsoft.com/en-us/microsoft-edge/enhance-your-security-on-the-web-with-microsoft-edge-b8199f13-b21b-4a08-a806-daed31a1929d) for details.
7. Virtual background requires [SharedArrayBuffer](/docs/video-sdk/web/sharedarraybuffer/) (SAB) for Firefox and Safari for WebAssembly video, but not for WebRTC video. Only supported on desktop browsers. For Chrome, use the [`enforceVirtualBackground`](https://marketplacefront.zoom.us/sdk/custom/web/interfaces/ZoomVideo.InitOptions.html#enforceVirtualBackground) `init` option to use virtual background without SAB.
8. WebRTC video supported on:
    - Chrome on Windows, macOS, Android (with some model-specific exceptions), iOS, and ChromeOS
    - Edge on Windows
    - Safari on macOS
    - Mobile Safari and other WebKit-based browsers on iOS
    - _**Support for additional browsers and OS platforms is being gradually expanded.**_
9. Share tab audio and share system audio require [SharedArrayBuffer](/docs/video-sdk/web/sharedarraybuffer/) for WebAssembly (WASM) audio, but not for WebRTC audio. When sharing with a Chrome or Edge browser, users can share a tab, window, or entire screen. If they share a tab, they can also share the tab audio. If they share a window or the entire screen, they can share the system audio. Developers control this with the [`systemAudio`](https://marketplacefront.zoom.us/sdk/custom/web/interfaces/ZoomVideo.ScreenShareOption.html#controls) option in `ZoomVideo.ScreenShareOption`. Non-Chromium browsers can only share the window or entire screen and microphone audio, but no computer audio as these browsers do not support audio capture in [`getDisplayMedia`](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getDisplayMedia#browser_compatibility).

## Check device or browser support

As browser vendors release new versions, the minimum supported browser version will increase up to two (2) versions behind the new version. For example:

-   If the current version of Chrome is 139, the SDK will support versions 137, 138, and 139.
-   When the current version increases to 140, the SDK will support versions 138, 139, and 140.

As a best practice, before calling the `init` function, we recommend that you check whether the device supports the core feature you are building for, like `video`, `audio`, or `screen` (screen sharing), by calling the `ZoomVideo.checkSystemRequirements()` function. For example:

```javascript
import ZoomVideo from "@zoom/videosdk";
const client = ZoomVideo.createClient();

let stream;

if (
    ZoomVideo.checkSystemRequirements().video &&
    ZoomVideo.checkSystemRequirements().audio
) {
    client
        .init("en-US", "Global", { patchJsMedia: true })
        .then(() => {
            client
                .join(topic, token, userName, password)
                .then(() => {
                    stream = client.getMediaStream();
                })
                .catch((error) => {
                    console.log(error);
                });
        })
        .catch((error) => {
            console.log(error);
        });
}
```

```jsx
import ZoomVideo from "@zoom/videosdk";
const client = ZoomVideo.createClient();

let stream;

if (
    ZoomVideo.checkSystemRequirements().video &&
    ZoomVideo.checkSystemRequirements().audio
) {
    await client.init("en-US", "Global", { patchJsMedia: true });

    await client.join(topic, token, userName, password);

    stream = client.getMediaStream();
}
```

## Mobile and tablet browser support

-   **iOS and iPadOS** - All browsers on iOS and iPadOS use the same underlying [WebKit](https://en.wikipedia.org/wiki/WebKit) engine (including Safari and Chrome). Because of this, supported features are based on the OS version, rather than individual browser versions.

-   **Android**

    -   Most Android browsers are based on [Chromium Blink](<https://en.wikipedia.org/wiki/Blink_(browser_engine)>) and generally follow its versioning.
    -   Samsung Internet is also Chromium-based but uses its [own versioning scheme](https://en.wikipedia.org/wiki/Samsung_Internet#History), which maps to specific Chromium versions.
    -   For simplicity, supported features are listed by Android OS version, as the SDK offers the same capabilities across all supported Android browsers.
    -   **Android Firefox is not supported** because it uses [GeckoView](<https://en.wikipedia.org/wiki/Gecko_(software)>), a different rendering engine, and different versioning.

-   **Microsoft Tablets** - Microsoft tablets run Windows, so browser support matches the same browsers on Windows desktops.

-   **Chromebooks** - Chromebooks run ChromeOS and support the same features as desktop browsers.

In general, mobile and tablet browser support follows OS versions currently supported by [Apple](https://en.wikipedia.org/wiki/IOS_version_history#Overview) and [Android](https://en.wikipedia.org/wiki/Android_version_history#Overview), including Samsung devices.

The Meeting SDK for web component view is designed specifically for desktop browser widget use cases and is not supported on mobile or tablet browsers. For mobile and tablet use cases, use [client view](/docs/meeting-sdk/web/client-view/).

## Content Security Policy header

If you're using Content Security Policy (CSP) on your browsers and want to use any Zoom SDK for web, you must configure your web server to return the Content-Security-Policy header. For example, using the [Nginx `add_header` directive](https://nginx.org/en/docs/http/ngx_http_headers_module.html).

```plaintext
add_header 'Content-Security-Policy' "default-src 'self';base-uri 'self';worker-src blob:;style-src 'self' 'unsafe-inline';script-src 'self' 'unsafe-inline' 'unsafe-eval' https://zoom.us *.zoom.us dmogdx0jrul3u.cloudfront.net blob:;connect-src 'self' https://zoom.us https://*.zoom.us wss://*.zoom.us;img-src 'self' https:;media-src 'self' https:;font-src 'self' https:;"
```

> **Warning**
>
> When `script-src-elem` is present in your CSP configuration, be sure its value matches the `script-src` directive.

For more on CSP, see the [Mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP), [web.dev](https://web.dev/csp/), or [W3C](https://www.w3.org/TR/CSP3/) documentation.

You'll need to add this if you see something like this error in the browser console.

```plaintext
CompileError: WebAssembly.instantiate(): Refused to compile or instantiate WebAssembly module because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "script-src 'self' 'strict-dynamic' 'nonce-LxlpktY6Sou9E-ZC46OlZA' blob: https: 'unsafe-inline'"
at 3afbd0c1-54af-4f01-9d3f-8ffb8dcb31cf:1:22144
```

Note: If your browser supports it, you can use `wasm-unsafe-eval` instead of `unsafe-eval`. However, not all browsers support this. See the mdn web docs for [CSP: script-src: Unsafe WebAssembly execution](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src#unsafe_webassembly_execution) for details.
