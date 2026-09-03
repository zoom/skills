<!-- Source: https://developers.zoom.us/docs/video-sdk/web/video-picture-in-picture.md -->
<!-- Fetched: 2026-06-15 -->

# Picture-in-picture


On Chromium browsers and Safari, you can configure the Video SDK for web to appear as a window on top of other browser content using picture-in-picture (PiP). Since the `video-player` used is not an `HTMLVideoElement`, it cannot directly enter PiP mode using the [`requestPictureInPicture()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLVideoElement/requestPictureInPicture) method. This example uses `window.documentPictureInPicture.requestWindow()` to create a PiP window, then adds the element to the window. Follow these steps.

1. **Check if PiP is supported.** Ensure that the [`documentPictureInPicture`](https://developer.mozilla.org/en-US/docs/Web/API/DocumentPictureInPicture) API is available and [`pictureInPictureEnabled`](https://developer.mozilla.org/en-US/docs/Web/API/Document/pictureInPictureEnabled) is `true`.
2. **Request PiP window.** Use [`requestWindow()`](https://developer.mozilla.org/en-US/docs/Web/API/DocumentPictureInPicture/requestWindow) to create a PiP window.
3. **Move the video element into the PiP window.** Transfer the element or its container to the PiP document.

## JavaScript example

```javascript
let pipWindow;
const enterPictureInPicture = async (videoContainer: VideoPlayerContainer) => {
  if (
    window.document.pictureInPictureEnabled &&
    "documentPictureInPicture" in window
  ) {
    try {
      pipWindow = await window.documentPictureInPicture.requestWindow();
      // Insert a stub after the container has been transferred to the PipWindow.
      const placeholder = document.createElement("div");
      placeholder.textContent =
        "The element has been transferred to the PipWindow";
      placeholder.id = "PiP-placeholder";
      if (videoContainer) {
        // Ensure styles are applied
        Array.from(document.styleSheets).forEach((styleSheet) => {
          Array.from(styleSheet.cssRules).forEach((rule) => {
            const style = document.createElement("style");
            style.textContent = rule.cssText;
            pipWindow.document.head.appendChild(style);
          });
        });
        // Move the video container into the PiP window
        videoContainer.parentNode?.insertBefore(placeholder, videoContainer);
        pipWindow.document.body.appendChild(videoContainer);
      }
    } catch (error) {
      console.error("Failed to enter Picture-in-Picture mode:", error);
    }
  } else {
    console.warn("Picture-in-Picture is not supported by your browser.");
  }
};

const leavePictureInPicture = async (videoContainer: VideoPlayerContainer) => {
  const placeholder = document.getElementById("PiP-placeholder");
  try {
    if (placeholder && videoContainer) {
      placeholder.parentNode?.replaceChild(videoContainer, placeholder);
      await pipWindow.close();
      pipWindow = undefined;
    }
  } catch (error) {
    console.error("Failed to leave Picture-in-Picture mode:", error);
  }
};
```

## Usage (React)

Call [`enterpictureinpicture`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLVideoElement/enterpictureinpicture_event) with the reference to your video container:

```tsx
const videoContainerRef = useRef<HTMLElement | null>(null);

// Example usage inside a button click handler
```

## From the developer blog

See the following blog post and sample for examples.

-   [Developing a picture-in-picture experience with Zoom Video SDK](/blog/video-sdk-picture-in-picture) by James Coon - 03-24-2024

-   [Document Picture-in-Picture example](https://github.com/mdn/dom-examples/tree/main/document-picture-in-picture)
