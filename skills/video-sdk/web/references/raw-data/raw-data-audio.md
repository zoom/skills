<!-- Source: https://developers.zoom.us/docs/video-sdk/web/raw-data-audio.md -->
<!-- Fetched: 2026-06-15 -->

# Video SDK - web - Raw data - audio


AudioProcessor is encapsulated based on the native web AudioWorkletProcessor. As a result, similar to the native web AudioWorkletProcessor, AudioProcessor runs within AudioWorkletGlobalScope to enhance performance. To define custom audio processing logic, extend the AudioProcessor interface. The following code declares the AudioProcessor globally to enable editor recognition.

```javascript
import type { AudioProcessor as SDKAudioProcessor, registerProcessor as SDKregisterProcessor } from '@zoom/videosdk';
declare global {
  /**
   * Abstract class that the custom audio processor needs to extend.
   */
  const AudioProcessor: typeof SDKAudioProcessor;
  /**
   * Registers a class constructor derived from AudioProcessor interface under a specified name.
   */
  const registerProcessor: typeof SDKregisterProcessor;
}
```

## Limitations

-   The SDK allows only one active audio processor of the same type at any given time.

## Required functions to override

The `AudioProcessor` class in the Video SDK is implemented as a subclass of the native JavaScript `AudioWorkletProcessor`, extending its functionality by introducing custom lifecycle management hooks (`onInit` and `onUninit`) to handle initialization and resource cleanup phases. This subclass adheres to the Web Audio API specification while augmenting the standard [AudioWorkletProcessor](https://developer.mozilla.org/docs/Web/API/AudioWorkletProcessor) workflow with framework-specific lifecycle control mechanisms.

-   `onInit` and `onUninit` are lifecycle functions triggered when the processor initializes or shuts down. Use these to allocate and release resources.

## Additional built-in functions

-   `registerProcessor` registers a class constructor derived from the AudioProcessor interface under a specified name.

When you register a processor, store an internal key-value pair in the format `{ name: constructor }` in the `AudioProcessor` worker global scope. The SDK uses the registered name when creating a processor instance.

## Example 1: White noise processor

```javascript
class WhiteNoiseProcessor extends AudioProcessor {
  constructor(port, options) {
    super(port, options);
    this.port.onmessage = (event) => {
      const { cmd, data } = event.data;
      console.log(`onmessage() cmd:${cmd}, data:${data}`);
    };
  }
  process(inputs: Array<Array<Float32Array>>, outputs: Array<Array<Float32Array>>) {
    const output = outputs[0];
    output.forEach((channel) => {
      for (let i = 0; i < channel.length; i++) {
        channel[i] = Math.random() * 2 - 1;
      }
    });
    return true;
  }
}

registerProcessor('white-noise-processor', WhiteNoiseProcessor);
```

## Example 2: Pitch shift processor

```javascript
class FamaleVoiceProcessor extends AudioProcessor {
  pitchRatio: number;
  bufferSize: number;
  buffer: Float32Array;
  writePos: number;
  readPos: number;
  formantRatio: number;
  dryWet: number;
  hpf: {
    prevIn: number;
    prevOut: number;
    alpha: number;
  };

  constructor(port: MessagePort, options: any) {
    super(port, options);
    this.bufferSize = 11025;
    this.buffer = new Float32Array(this.bufferSize);
    this.writePos = 0;
    this.readPos = 0.0;
    this.pitchRatio = 1.5;
    this.formantRatio = 1.2;
    this.dryWet = 0.7;
    this.hpf = {
      prevIn: 0,
      prevOut: 0,
      alpha: 0.86
    };
  }

  process(inputs: Array<Array<Float32Array>>, outputs: Array<Array<Float32Array>>) {
    const input = inputs[0];
    const output = outputs[0];
    if (input.length === 0 || !input[0]) {
      return true;
    }
    const inputChannel = input[0];
    const outputChannel = output[0];
    for (let i = 0; i < inputChannel.length; i++) {
      this.buffer[this.writePos] = inputChannel[i];
      this.writePos = (this.writePos + 1) % this.bufferSize;
    }
    for (let i = 0; i < outputChannel.length; i++) {
      let readPos = this.readPos % this.bufferSize;
      if (readPos < 0) readPos += this.bufferSize;
      const intPos = Math.floor(readPos);
      const frac = readPos - intPos;
      const nextPos = (intPos + 1) % this.bufferSize;
      const raw = this.buffer[intPos] * (1 - frac) + this.buffer[nextPos] * frac;
      const filtered = raw - this.hpf.prevIn + this.hpf.alpha * this.hpf.prevOut;
      this.hpf.prevIn = raw;
      this.hpf.prevOut = filtered;
      outputChannel[i] = filtered * this.dryWet + raw * (1 - this.dryWet);
      this.readPos += this.pitchRatio;
      if (this.readPos >= this.bufferSize) {
        this.readPos -= this.bufferSize;
        this.writePos = 0;
      }
    }
    return true;
  }
}

registerProcessor('white-noise-audio-processor', FamaleVoiceProcessor);
```

> **Note**
>
> This is a basic implementation. Professional voice changing requires more complex signal processing. The ideal effect requires combining formant adjustment and more precise filtering.

## Create processor and add to the audio pipeline

Create a processor instance and add it to the audio pipeline.

### Create a processor instance

Use `stream.createProcessor` to create a processor instance. The `url`, which specifies the script location, must either originate from the same domain or have the appropriate CORS headers.

```javascript
const params = {
    name: "white-noise-processor",
    type: "audio",
    url: "[absolute url of processor script]",
    options: {},
};
const processor = await stream.createProcessor(params);
```

### Add processor to audio stream pipeline

Once created, add the processor to the audio stream pipeline using `stream.addProcessor(processor)`. You can perform this operation before or after starting the audio.

```javascript
// Add a processor
await stream.addProcessor(processor);
// Update the parameters
processor.port?.postMessage({
    cmd: "update_audio_type",
    data,
});
// Remove a processor
await stream.removeProcessor(processor);
```

## Samples

See the samples for examples of simple implementations.

-   [Zoom Media Processor Sample](https://github.com/zoom/videosdk-web-processor-sample)
