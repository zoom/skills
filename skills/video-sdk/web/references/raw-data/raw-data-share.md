<!-- Source: https://developers.zoom.us/docs/video-sdk/web/raw-data-share.md -->
<!-- Fetched: 2026-06-15 -->

# Video SDK - web - Raw data - share


To define custom screen share processing logic, extend the `ShareProcessor` interface. `ShareProcessor` is a preprocessor that only processes raw screen share data. It requires `VideoFrame` and `OffscreenCanvas` API support. The following code declares the `ShareProcessor` globally to enable editor recognition.

```typescript
import type {
    ShareProcessor as SDKShareProcessor,
    registerProcessor as SDKregisterProcessor,
} from "@zoom/videosdk";
declare global {
    /**
     * Abstract class that the custom share processor needs to extend.
     */
    const ShareProcessor: typeof SDKShareProcessor;
    /**
     * Registers a class constructor derived from ShareProcessor interface under a specified name.
     */
    const registerProcessor: typeof SDKregisterProcessor;
}
```

## Current limitation

-   The SDK allows only one active share processor of the same type at any given time.

## Required functions to override

Custom share processors must override the following functions.

-   `constructor` - Accepts a `port` parameter for message communication between threads and an optional `options` object for initial parameters.
-   `processFrame` - Defines how to process each frame. The SDK calls this method for every video frame, taking a `VideoFrame` as input and modifying the `OffscreenCanvas` output. The method's return value determines whether the SDK applies the effect to the frame sent remotely. Return `true` to process the frame and apply the effect to the one sent remotely. Return `false` to process the frame without affecting the one sent remotely.
-   `onInit` and `onUninit` - Lifecycle functions triggered when the processor initializes or shuts down. Use these to allocate and release resources.

## Additional built-in functions

-   `getOutput()` - Returns the `OffscreenCanvasoutput`, which is the same as the second parameter of `processFrame()`.
-   `registerProcessor` - Registers a class constructor derived from the `ShareProcessor` interface under a specified name.

> **Note**
>
> When you register a processor, store an internal key-value pair in the format `{ name: constructor }` in the `ShareProcessor` worker global scope. The SDK uses the registered name when creating a processor instance.

### Example: PII mask processor

```javascript
/**
 * PiiWebGLShareProcessor.ts
 * A ShareProcessor that applies a Gaussian blur to a specified normalized region
 * of each VideoFrame using WebGL2 for high performance.
 */
interface BlurRegion {
  x: number;
  y: number;
  width: number;
  height: number;
}
interface ProcessorOptions {
  blurRegionNorm?: BlurRegion;
  blurRadius?: number;
}
interface MessageEventData {
  command: string;
  data: {
    blurRegionNorm?: BlurRegion;
    blurRadius?: number;
  };
}
interface UniformLocations {
  uW: WebGLUniformLocation | null;
  uO: WebGLUniformLocation | null;
  uWV: WebGLUniformLocation | null;
  uOV: WebGLUniformLocation | null;
  uR: WebGLUniformLocation | null;
  uOrig: WebGLUniformLocation | null;
  uBlur: WebGLUniformLocation | null;
}
interface WebGLResources {
  gl: WebGL2RenderingContext | null;
  texOrig: WebGLTexture | null;
  tex: WebGLTexture | null;
  tex2: WebGLTexture | null;
  fbo: WebGLFramebuffer | null;
  fbo2: WebGLFramebuffer | null;
  vao: WebGLVertexArrayObject | null;
  progH: WebGLProgram | null;
  progV: WebGLProgram | null;
  progC: WebGLProgram | null;
}
// Vertex shader: full‐screen quad
const VERTEX_SHADER_SOURCE = `#version 300 es
in vec2 a_position;
in vec2 a_texCoord;
out vec2 v_uv;

void main() {
  v_uv = a_texCoord;
  gl_Position = vec4(a_position, 0.0, 1.0);
}`;

// Fragment shader: horizontal Gaussian blur
const HORIZONTAL_BLUR_FRAGMENT_SHADER = `#version 300 es
precision mediump float;
in vec2 v_uv;
uniform sampler2D u_texture;
uniform float u_texelOffset;
uniform float u_weights[9];
out vec4 outColor;

void main() {
  vec2 off = vec2(u_texelOffset, 0.0);
  vec4 sum = texture(u_texture, v_uv) * u_weights[0];
  for (int i = 1; i < 9; ++i) {
    sum += texture(u_texture, v_uv + off * float(i)) * u_weights[i];
    sum += texture(u_texture, v_uv - off * float(i)) * u_weights[i];
  }
  outColor = sum;
}`;

// Fragment shader: vertical Gaussian blur
const VERTICAL_BLUR_FRAGMENT_SHADER = `#version 300 es
precision mediump float;
in vec2 v_uv;
uniform sampler2D u_texture;
uniform float u_texelOffset;
uniform float u_weights[9];
out vec4 outColor;

void main() {
  vec2 off = vec2(0.0, u_texelOffset);
  vec4 sum = texture(u_texture, v_uv) * u_weights[0];
  for (int i = 1; i < 9; ++i) {
    sum += texture(u_texture, v_uv + off * float(i)) * u_weights[i];
    sum += texture(u_texture, v_uv - off * float(i)) * u_weights[i];
  }
  outColor = sum;
}`;

// Fragment shader: composite original and blurred based on region
const COMPOSITE_FRAGMENT_SHADER = `#version 300 es
precision mediump float;
in vec2 v_uv;
uniform sampler2D u_orig;
uniform sampler2D u_blur;
uniform vec4 u_region;
out vec4 outColor;

void main() {
  if (v_uv.x >= u_region.x && v_uv.x <= u_region.x + u_region.z &&
      v_uv.y >= u_region.y && v_uv.y <= u_region.y + u_region.w) {
    outColor = texture(u_blur, v_uv);
  } else {
    outColor = texture(u_orig, v_uv);
  }
}`;

function compileShader(gl: WebGL2RenderingContext, type: number, source: string): WebGLShader {
  const shader = gl.createShader(type);
  if (!shader) {
    throw new Error('Failed to create shader');
  }
  gl.shaderSource(shader, source);
  gl.compileShader(shader);
  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
    const error = gl.getShaderInfoLog(shader);
    gl.deleteShader(shader);
    throw new Error(`Shader compilation failed: ${error}`);
  }
  return shader;
}

function createProgram(gl: WebGL2RenderingContext, vertexSource: string, fragmentSource: string): WebGLProgram {
  const program = gl.createProgram();
  if (!program) {
    throw new Error('Failed to create program');
  }
  const vertexShader = compileShader(gl, gl.VERTEX_SHADER, vertexSource);
  const fragmentShader = compileShader(gl, gl.FRAGMENT_SHADER, fragmentSource);
  gl.attachShader(program, vertexShader);
  gl.attachShader(program, fragmentShader);
  gl.linkProgram(program);
  if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
    const error = gl.getProgramInfoLog(program);
    gl.deleteProgram(program);
    gl.deleteShader(vertexShader);
    gl.deleteShader(fragmentShader);
    throw new Error(`Program linking failed: ${error}`);
  }
  gl.deleteShader(vertexShader);
  gl.deleteShader(fragmentShader);
  return program;
}

function buildGaussianWeights(sigma: number): Float32Array {
  const weights: number[] = [];
  let sum = 0;
  const twoSigmaSq = 2 * sigma * sigma;
  for (let i = 0; i < 9; i++) {
    const val = Math.exp(-(i * i) / twoSigmaSq);
    weights.push(val);
    sum += i === 0 ? val : val * 2;
  }
  return new Float32Array(weights.map((x) => x / sum));
}
class PiiWebGLProcessor extends ShareProcessor implements WebGLResources, UniformLocations {
  public gl: WebGL2RenderingContext | null = null;
  public texOrig: WebGLTexture | null = null;
  public tex: WebGLTexture | null = null;
  public tex2: WebGLTexture | null = null;
  public fbo: WebGLFramebuffer | null = null;
  public fbo2: WebGLFramebuffer | null = null;
  public vao: WebGLVertexArrayObject | null = null;
  public progH: WebGLProgram | null = null;
  public progV: WebGLProgram | null = null;
  public progC: WebGLProgram | null = null;
  public uW: WebGLUniformLocation | null = null;
  public uO: WebGLUniformLocation | null = null;
  public uWV: WebGLUniformLocation | null = null;
  public uOV: WebGLUniformLocation | null = null;
  public uR: WebGLUniformLocation | null = null;
  public uOrig: WebGLUniformLocation | null = null;
  public uBlur: WebGLUniformLocation | null = null;
  private _inited: boolean = false;
  private weights: Float32Array | null = null;
  private region: BlurRegion;
  private radius: number;
  constructor(port: MessagePort, options: ProcessorOptions = {}) {
    super(port, options);
    port.onmessage = (event: MessageEvent<MessageEventData>): void => {
      try {
        const { command, data } = event.data;
        if (command === 'update-blur-options') {
          if (data.blurRegionNorm) {
            this.region = data.blurRegionNorm;
          }
          if (typeof data.blurRadius === 'number' && data.blurRadius > 0) {
            this.radius = data.blurRadius;
            this.weights = buildGaussianWeights(this.radius);
          }
        }
      } catch (error) {
        console.error('Error processing message:', error);
      }
    };
    this.region = options.blurRegionNorm || {
      x: 0.2,
      y: 0.2,
      width: 0.6,
      height: 0.6
    };
    this.radius = this.validateRadius(options.blurRadius);
  }
  private validateRadius(radius?: number): number {
    if (typeof radius === 'number' && radius > 0) {
      return radius;
    }
    return 10;
  }
  public onInit(): void {}
  public onUninit(): void {
    this.cleanupWebGLResources();
    this.resetState();
  }
  private cleanupWebGLResources(): void {
    if (!this.gl || this.gl.isContextLost()) {
      return;
    }
    const gl = this.gl;
    this.deleteTexture(gl, this.texOrig);
    this.deleteTexture(gl, this.tex);
    this.deleteTexture(gl, this.tex2);
    this.deleteFramebuffer(gl, this.fbo);
    this.deleteFramebuffer(gl, this.fbo2);
    if (this.vao) {
      gl.deleteVertexArray(this.vao);
    }
    this.deleteProgram(gl, this.progH);
    this.deleteProgram(gl, this.progV);
    this.deleteProgram(gl, this.progC);
    gl.bindFramebuffer(gl.FRAMEBUFFER, null);
    gl.bindTexture(gl.TEXTURE_2D, null);
    gl.bindVertexArray(null);
    gl.useProgram(null);
  }
  private deleteTexture(gl: WebGL2RenderingContext, texture: WebGLTexture | null): void {
    if (texture) {
      gl.deleteTexture(texture);
    }
  }
  private deleteFramebuffer(gl: WebGL2RenderingContext, framebuffer: WebGLFramebuffer | null): void {
    if (framebuffer) {
      gl.deleteFramebuffer(framebuffer);
    }
  }
  private deleteProgram(gl: WebGL2RenderingContext, program: WebGLProgram | null): void {
    if (program) {
      gl.deleteProgram(program);
    }
  }
  private resetState(): void {
    this.gl = null;
    this.texOrig = null;
    this.tex = null;
    this.tex2 = null;
    this.fbo = null;
    this.fbo2 = null;
    this.vao = null;
    this.progH = null;
    this.progV = null;
    this.progC = null;
    this.uW = null;
    this.uO = null;
    this.uWV = null;
    this.uOV = null;
    this.uR = null;
    this.uOrig = null;
    this.uBlur = null;
    this._inited = false;
    this.weights = null;
  }
  public async processFrame(input: VideoFrame, output: OffscreenCanvas): Promise<boolean> {
    try {
      const width = input.codedWidth;
      const height = input.codedHeight;
      output.width = width;
      output.height = height;
      if (!this._inited) {
        this.initializeGL(output, width, height);
        this._inited = true;
      }
      if (!this.gl || this.gl.isContextLost()) {
        throw new Error('WebGL context is not available or lost');
      }
      const gl = this.gl;
      gl.viewport(0, 0, width, height);
      if (!this.texOrig || !this.vao || !this.weights) {
        throw new Error('WebGL resources not properly initialized');
      }
      gl.bindTexture(gl.TEXTURE_2D, this.texOrig);
      gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, input);
      gl.bindVertexArray(this.vao);
      this.performHorizontalBlur(gl, width);
      this.performVerticalBlur(gl, height);
      this.performComposite(gl);
      gl.bindVertexArray(null);
      input.close();

      return true;
    } catch (error) {
      console.error('Error processing frame:', error);
      input.close();
      return false;
    }
  }

  private performHorizontalBlur(gl: WebGL2RenderingContext, width: number): void {
    if (!this.fbo || !this.progH || !this.uW || !this.uO || !this.weights) {
      throw new Error('Horizontal blur resources not initialized');
    }
    gl.bindFramebuffer(gl.FRAMEBUFFER, this.fbo);
    gl.useProgram(this.progH);
    gl.uniform1fv(this.uW, this.weights);
    gl.uniform1f(this.uO, this.radius / 8.0 / width);
    gl.activeTexture(gl.TEXTURE0);
    gl.bindTexture(gl.TEXTURE_2D, this.texOrig);
    gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);
  }
  private performVerticalBlur(gl: WebGL2RenderingContext, height: number): void {
    if (!this.fbo2 || !this.progV || !this.uWV || !this.uOV || !this.weights || !this.tex) {
      throw new Error('Vertical blur resources not initialized');
    }
    gl.bindFramebuffer(gl.FRAMEBUFFER, this.fbo2);
    gl.useProgram(this.progV);
    gl.uniform1fv(this.uWV, this.weights);
    gl.uniform1f(this.uOV, this.radius / 8.0 / height);
    gl.activeTexture(gl.TEXTURE0);
    gl.bindTexture(gl.TEXTURE_2D, this.tex);
    gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);
  }
  private performComposite(gl: WebGL2RenderingContext): void {
    if (!this.progC || !this.uR || !this.uOrig || !this.uBlur || !this.tex2) {
      throw new Error('Composite resources not initialized');
    }
    gl.bindFramebuffer(gl.FRAMEBUFFER, null);
    gl.useProgram(this.progC);
    gl.uniform4f(this.uR, this.region.x, this.region.y, this.region.width, this.region.height);
    gl.uniform1i(this.uOrig, 0);
    gl.uniform1i(this.uBlur, 1);
    gl.activeTexture(gl.TEXTURE0);
    gl.bindTexture(gl.TEXTURE_2D, this.texOrig);
    gl.activeTexture(gl.TEXTURE1);
    gl.bindTexture(gl.TEXTURE_2D, this.tex2);
    gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);
  }
  private initializeGL(canvas: OffscreenCanvas, width: number, height: number): void {
    const gl = canvas.getContext('webgl2');
    if (!gl) {
      throw new Error('Unable to get WebGL2 context');
    }
    this.gl = gl;
    gl.viewport(0, 0, width, height);
    this.weights = buildGaussianWeights(this.radius);
    this.createShaderPrograms(gl);
    this.getUniformLocations(gl);
    this.initializeTextures(gl, width, height);
    this.setupVertexArrayObject(gl);
  }
  private createShaderPrograms(gl: WebGL2RenderingContext): void {
    try {
      this.progH = createProgram(gl, VERTEX_SHADER_SOURCE, HORIZONTAL_BLUR_FRAGMENT_SHADER);
      this.progV = createProgram(gl, VERTEX_SHADER_SOURCE, VERTICAL_BLUR_FRAGMENT_SHADER);
      this.progC = createProgram(gl, VERTEX_SHADER_SOURCE, COMPOSITE_FRAGMENT_SHADER);
    } catch (error) {
      throw new Error(`Failed to create shader programs: ${error}`);
    }
  }
  private getUniformLocations(gl: WebGL2RenderingContext): void {
    if (!this.progH || !this.progV || !this.progC) {
      throw new Error('Shader programs not initialized');
    }
    this.uW = gl.getUniformLocation(this.progH, 'u_weights');
    this.uO = gl.getUniformLocation(this.progH, 'u_texelOffset');
    this.uWV = gl.getUniformLocation(this.progV, 'u_weights');
    this.uOV = gl.getUniformLocation(this.progV, 'u_texelOffset');
    this.uR = gl.getUniformLocation(this.progC, 'u_region');
    this.uOrig = gl.getUniformLocation(this.progC, 'u_orig');
    this.uBlur = gl.getUniformLocation(this.progC, 'u_blur');
  }
  private initializeTextures(gl: WebGL2RenderingContext, width: number, height: number): void {
    this.texOrig = this.createTexture(gl);
    this.configureTexture(gl, this.texOrig);
    this.tex = this.createTexture(gl);
    this.configureTexture(gl, this.tex);
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, width, height, 0, gl.RGBA, gl.UNSIGNED_BYTE, null);
    this.fbo = gl.createFramebuffer();
    if (!this.fbo) {
      throw new Error('Failed to create framebuffer');
    }
    gl.bindFramebuffer(gl.FRAMEBUFFER, this.fbo);
    gl.framebufferTexture2D(gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, this.tex, 0);
    this.tex2 = this.createTexture(gl);
    this.configureTexture(gl, this.tex2);
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, width, height, 0, gl.RGBA, gl.UNSIGNED_BYTE, null);
    this.fbo2 = gl.createFramebuffer();
    if (!this.fbo2) {
      throw new Error('Failed to create framebuffer 2');
    }
    gl.bindFramebuffer(gl.FRAMEBUFFER, this.fbo2);
    gl.framebufferTexture2D(gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, this.tex2, 0);
    gl.bindFramebuffer(gl.FRAMEBUFFER, null);
    gl.bindTexture(gl.TEXTURE_2D, null);
  }
  private createTexture(gl: WebGL2RenderingContext): WebGLTexture {
    const texture = gl.createTexture();
    if (!texture) {
      throw new Error('Failed to create texture');
    }
    return texture;
  }
  private configureTexture(gl: WebGL2RenderingContext, texture: WebGLTexture): void {
    gl.bindTexture(gl.TEXTURE_2D, texture);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
  }
  private setupVertexArrayObject(gl: WebGL2RenderingContext): void {
    if (!this.progH) {
      throw new Error('Horizontal blur program not initialized');
    }
    const vertices = new Float32Array([-1, -1, 0, 1, 1, -1, 1, 1, -1, 1, 0, 0, 1, 1, 1, 0]);
    this.vao = gl.createVertexArray();
    if (!this.vao) {
      throw new Error('Failed to create vertex array object');
    }
    const buffer = gl.createBuffer();
    if (!buffer) {
      throw new Error('Failed to create vertex buffer');
    }
    gl.bindVertexArray(this.vao);
    gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
    gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
    const positionLocation = gl.getAttribLocation(this.progH, 'a_position');
    if (positionLocation !== -1) {
      gl.enableVertexAttribArray(positionLocation);
      gl.vertexAttribPointer(positionLocation, 2, gl.FLOAT, false, 16, 0);
    }
    const texCoordLocation = gl.getAttribLocation(this.progH, 'a_texCoord');
    if (texCoordLocation !== -1) {
      gl.enableVertexAttribArray(texCoordLocation);
      gl.vertexAttribPointer(texCoordLocation, 2, gl.FLOAT, false, 16, 8);
    }
    gl.bindVertexArray(null);
  }
}

registerProcessor('share-webgl-pii-processor', PiiWebGLProcessor);
```

## Create processor and add to the share pipeline

Create a processor instance and add it to the share pipeline.

### Create a processor instance

Use `stream.createProcessor` to create a processor instance. The `url`, which specifies the script location, must either originate from the same domain or have the appropriate CORS headers.

```typescript
const params = {
    name: "share-webgl-pii-processor",
    type: "share",
    url: "[absolute url of processor script]",
    options: {
        blurRegionNorm: {
            x: 0.2,
            y: 0.2,
            width: 0.35,
            height: 0.3,
        },
        blurRadius: 50,
    },
};
const processor = await stream.createProcessor(params);
```

### Add processor to share stream pipeline

Once created, add the processor to the share stream pipeline using `stream.addProcessor(processor)`. You can perform this operation before or after starting the share.

```typescript
// Add a processor
await stream.addProcessor(processor);
// Update the parameters
processor.port?.postMessage({
    cmd: "update-blur-options",
    data: {
        blurRegionNorm: {
            x: 0.2,
            y: 0.2,
            width: 0.35,
            height: 0.3,
        },
        blurRadius: 50,
    },
});
// Remove a processor
await stream.removeProcessor(processor);
```

## Samples

See the samples for examples of simple implementations.

-   [Zoom Media Processor Sample](https://github.com/zoom/videosdk-web-processor-sample)
