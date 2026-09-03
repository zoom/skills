<!-- Source: https://developers.zoom.us/docs/video-sdk/web/raw-data-video.md -->
<!-- Fetched: 2026-06-15 -->

# Video SDK - web - Raw data - video


The video processor runs within a web worker to enhance performance. To define custom video processing logic, extend the `VideoProcessor` interface. The following code declares the `VideoProcessor` globally to enable editor recognition.

```javascript
import type { VideoProcessor as SDKVideoProcessor, registerProcessor as SDKregisterProcessor } from '@zoom/videosdk';
declare global {
  /**
   * Abstract class that the custom video processor needs to extend.
   */
  const VideoProcessor: typeof SDKVideoProcessor;
  /**
   * Registers a class constructor derived from VideoProcessor interface under a specified name.
   */
  const registerProcessor: typeof SDKregisterProcessor;
}
```

## Current limitations

-   You can't use the processor simultaneously with virtual backgrounds or video masks.
-   The SDK allows only one active video processor of the same type at any given time.

## Required functions to override

To define custom video processing logic, you must override the following functions in the class.

-   `constructor` - Accepts a `port` parameter for message communication between threads and an optional `options` object for initial parameters.
-   `processFrame` - Defines how to process each video frame. The SDK calls this method for every video frame, taking a `VideoFrame` as input and modifying the `OffscreenCanvas` output. The method's return value determines whether the SDK applies the effect to the frame sent remotely. Return `true` to process the frame and apply the effect to the one sent remotely. Return `false` to process the frame without affecting the one sent remotely.
-   `onInit` and `onUninit`: Lifecycle functions triggered when the processor initializes or shuts down. Use these to allocate and release resources.

## Additional built-in functions

-   `getOutput()` - Returns the `OffscreenCanvas` output, which is the same as the second parameter of `processFrame()`.
-   `registerProcessor` - Registers a class constructor derived from the `VideoProcessor` interface under a specified name.

> **Note:** When you register a processor, store an internal key-value pair in the format `{ name: constructor }` in the `VideoProcessor` worker global scope. The SDK uses the registered name when creating a processor instance.

## Example: Watermark processor

This watermark example builds a video processor to overlay a watermark image on top of each video frame. Define a video processor by extending the `VideoProcessor` interface. Define a `context` field to store the canvas context and a `watermarkImage` field to store the image overlay.

```javascript
class WatermarkProcessor extends VideoProcessor {
  private context: OffscreenCanvasRenderingContext2D | null = null;
  private watermarkImage: ImageBitmap | null = null;

  // The constructor initializes the processor and sets up the message listener.
  // Listen for an update_watermark_image event and update the watermark image from the message.
  constructor(port: MessagePort, options?: any) {
    super(port, options);
    port.addEventListener('message', (e) => {
      if (e.data.cmd === 'update_watermark_image') {
        this.updateWatermarkImage(e.data.data);
      }
    });
  }

  // The processFrame function is called for every video frame.
  // Use this to modify the video frame.
  async processFrame(input: VideoFrame, output: OffscreenCanvas) {
    this.renderFrame(input, output);
    return true;
  }

  // The onInit function is called when the processor is initialized.
  // Use this to access the output canvas and initialize the context.
  onInit() {
    const canvas = this.getOutput();
    if (canvas) {
      this.context = canvas.getContext('2d');
      if (!this.context) {
        console.error('2D context could not be initialized.');
      }
    }
  }

  // The onUninit function is called when the processor is uninitialized.
  // Use this to clean up resources.
  onUninit() {
    this.context = null;
    this.watermarkImage = null;
  }

  private updateWatermarkImage(image: ImageBitmap) {
    this.watermarkImage = image;
  }

  private renderFrame(input: VideoFrame, output: OffscreenCanvas) {
    if (!this.context) return;

    // Draw the video frame onto the canvas by calling the
    // [drawImage](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage) method.
    // It takes the input image, the x and y coordinates of the top-left corner, and the width and height of the image.
    this.context.drawImage(input, 0, 0, output.width, output.height);

    // Draw the watermark image on top of the input frame.
    // Use the [globalAlpha](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/globalAlpha) property
    // to set the transparency of the watermark image to 50%.
    // Call the drawImage method to draw the watermark image on top of the input frame.
    // Set the opacity with the globalAlpha property.
    if (this.watermarkImage) {
      const watermarkWidth = this.watermarkImage.width;
      const watermarkHeight = this.watermarkImage.height;
      this.context.globalAlpha = 0.5;
      this.context.drawImage(this.watermarkImage, 0, 0, watermarkWidth, watermarkHeight);
      this.context.globalAlpha = 1.0;
    }
  }
}

// Register the processor class with the SDK by calling the registerProcessor function
// with the processor name and the processor class.
registerProcessor('watermark-processor', WatermarkProcessor);
```

## Add processor to video pipeline

Create a processor instance and add it to the video pipeline.

### Create a processor instance

Use `stream.createProcessor` to create a processor instance. Pass in a `name` for the processor and the `type` of the processor. The `url` specifies the script location; it must originate from the same domain or have the appropriate CORS headers. The following code continues with the watermark example.

```typescript
const client = ZoomVideo.createClient();
const mediaStream = client.getMediaStream();

const processor = await mediaStream.createProcessor({
    name: "watermark-processor",
    type: "video",
    url: window.location.origin + "/watermark-processor.js",
});
```

### Add processor to video stream pipeline

Once created, add the processor to the video stream pipeline using `stream.addProcessor(processor)`. You can perform this operation before or after starting the video.

```javascript
// Add a processor
await stream.addProcessor(processor);
```

Use the following to remove a processor.

```javascript
// Remove a processor
await stream.removeProcessor(processor);
```

### Apply the watermark

To finish the watermark example, you must send an image bitmap to the watermark processor. This example uses [`fabric.js`](https://fabricjs.com/) to create a bitmap image of the text in red and return it as an [`ImageBitmap`](https://developer.mozilla.org/en-US/docs/Web/API/ImageBitmap) object.

```typescript
import { Canvas, FabricText } from "fabric";

export async function getBitmap(message: string) {
    const canvas = new Canvas(document.createElement("canvas"), {
        width: 1280,
        height: 720,
    });
    const text = new FabricText(message, {
        left: 10,
        top: 10,
        fontSize: 60,
        fill: "red",
    });
    canvas.add(text);
    canvas.renderAll();
    const dataUrl = canvas.toDataURL();
    const response = await fetch(dataUrl);
    const blob = await response.blob();
    const imageBitmap = await createImageBitmap(blob);
    return imageBitmap;
}
```

Call the `getBitmap` function and pass the image bitmap data to the video processor using the `postMessage` method.

```typescript
const imageBitmap = await getBitmap("hello world");
processor.port.postMessage({
    cmd: "update_watermark_image",
    image: imageBitmap,
});
```

This renders red text reading "hello world" on the top left of each video frame of the user's video. This is visible to all other remote users as well. You can modify the text/image by calling the postMessage method with the new image bitmap:

```typescript
// later in the app
processor.port.postMessage({
    cmd: "update_watermark_image",
    image: newImageBitmap,
});
```

That's all the code you need to get a basic watermark working.

## Samples

See the samples for examples of simple implementations.

-   [Zoom Media Processor Sample](https://github.com/zoom/videosdk-web-processor-sample)
-   [Video SDK Web Video-Processor Quickstart](https://github.com/zoom/videosdk-web-videoprocessor-quickstart/)

## From the blog

-   [Customize Video SDK streams with watermarks](/blog/customize-video-stream-with-watermarks) by Ekaansh Arora - 08-28-2025
