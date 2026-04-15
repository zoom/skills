# Advanced Features

## Chat

In-session text messaging with file sharing support.

### Setup

```typescript
const chat = client.getChatClient();
```

### Send Messages

```typescript
// Send to everyone
await chat.sendToAll('Hello everyone!');

// Send to specific user
await chat.send('Private message', userId);
```

### Message Limits

- **Max length**: 10,000 bytes (recommended: < 1,000 characters)
- Messages exceeding limit are truncated

### Receive Messages

```typescript
client.on('chat-on-message', (payload) => {
  const {
    message,        // Message content
    sender,         // { userId, name }
    receiver,       // { userId, name } or 'everyone'
    timestamp,      // Unix timestamp
    id,             // Message ID
  } = payload;

  console.log(`${sender.name}: ${message}`);
});
```

### Chat History

```typescript
// Get messages from current session
const history = chat.getHistory();
// Only includes messages sent to everyone
```

### Chat Privileges

```typescript
// Get current privilege
const privilege = chat.getPrivilege();

// Set privilege (host only)
await chat.setPrivilege(ChatPrivilege.All);          // Everyone can chat
await chat.setPrivilege(ChatPrivilege.NoOne);        // No one can chat
await chat.setPrivilege(ChatPrivilege.EveryonePublicly); // Public only
```

### File Sharing

```typescript
// Send file
await chat.sendFile(file, userId); // or omit userId for everyone

// Receive file
client.on('chat-on-message', (payload) => {
  if (payload.file) {
    const { name, size, downloadUrl } = payload.file;
    // Handle file download
  }
});

// Upload progress
client.on('chat-file-upload-progress', (payload) => {
  const { progress, status } = payload;
  // ChatFileUploadStatus: Init | InProgress | Success | Fail | Cancel | Complete
});

// Download progress
client.on('chat-file-download-progress', (payload) => {
  const { progress, status } = payload;
});
```

### Chat Events

```typescript
// Privilege changed
client.on('chat-privilege-change', (payload) => {
  const { privilege } = payload;
});
```

---

## Whiteboard

Collaborative whiteboard for drawing and annotation.

### Setup

```typescript
const whiteboard = client.getWhiteboardClient();
```

### Start Whiteboard

```typescript
// Check if can start
const canStart = whiteboard.canStartWhiteboard();

// Start whiteboard
await whiteboard.startWhiteboard();
```

### Stop Whiteboard

```typescript
await whiteboard.stopWhiteboard();
```

### Whiteboard Events

```typescript
// Status change
client.on('whiteboard-status-change', (payload) => {
  const { status, oderId } = payload;
  // status: 'Active' | 'Inactive'
});

// Peer whiteboard state
client.on('peer-whiteboard-state-change', (payload) => {
  const { userId, action } = payload;
  // action: 'Start' | 'Stop'
});
```

---

## Custom Processors

Process audio/video/share streams with custom logic.

### Video Processor

Create a worker file `video-processor.js`:

```javascript
// video-processor.js
class MyVideoProcessor extends VideoProcessor {
  constructor(port, options) {
    super(port, options);
  }

  onInit() {
    console.log('Video processor initialized');
  }

  processFrame(input, output) {
    const ctx = output.getContext('2d');

    // Draw input frame
    ctx.drawImage(input, 0, 0);

    // Apply custom effects
    // ...

    return true;
  }

  onUninit() {
    console.log('Video processor destroyed');
  }
}

registerProcessor('my-video-processor', MyVideoProcessor);
```

### Use Video Processor

```typescript
// Create processor
const processor = await stream.createProcessor({
  url: '/path/to/video-processor.js',
  name: 'my-video-processor',
  type: 'video',
  options: { /* custom options */ }
});

// Communicate with processor
processor.port.postMessage({ type: 'config', data: {} });

processor.port.onmessage = (event) => {
  console.log('From processor:', event.data);
};

// Destroy processor
await stream.destroyProcessor(processor);
```

### Audio Processor

Create a worker file `audio-processor.js`:

```javascript
// audio-processor.js (AudioWorklet)
class MyAudioProcessor extends AudioProcessor {
  constructor(port, options) {
    super(port, options);
  }

  process(inputs, outputs, parameters) {
    const input = inputs[0];
    const output = outputs[0];

    // Process audio samples
    for (let channel = 0; channel < input.length; channel++) {
      const inputChannel = input[channel];
      const outputChannel = output[channel];

      for (let i = 0; i < inputChannel.length; i++) {
        // Apply custom processing
        outputChannel[i] = inputChannel[i];
      }
    }

    return true;
  }
}

registerProcessor('my-audio-processor', MyAudioProcessor);
```

### Use Audio Processor

```typescript
const processor = await stream.createProcessor({
  url: '/path/to/audio-processor.js',
  name: 'my-audio-processor',
  type: 'audio',
  options: {}
});
```

### Share Processor

```javascript
// share-processor.js
class MyShareProcessor extends ShareProcessor {
  processFrame(input, output) {
    const ctx = output.getContext('2d');
    ctx.drawImage(input, 0, 0);
    // Add watermark, effects, etc.
    return true;
  }
}

registerProcessor('my-share-processor', MyShareProcessor);
```

### Processor Interface

```typescript
interface ProcessorParams {
  url: string;      // Worker script URL (same origin or CORS)
  name: string;     // Processor name
  type: 'audio' | 'video' | 'share';
  options?: any;    // Custom options passed to constructor
}

interface Processor {
  name: string;
  type: 'audio' | 'video' | 'share';
  port: MessagePort;  // Communication channel
}
```

---

## Media File Input

Use media files as audio/video input.

### Video File Input

```typescript
await stream.startVideo({
  mediaFile: {
    url: 'https://example.com/video.mp4',
    currentTime: 0,    // Start position in seconds
    loop: true,        // Loop playback
  }
});
```

### Audio File Input

```typescript
await stream.startAudio({
  mediaFile: {
    url: 'https://example.com/audio.mp3',
    currentTime: 0,
    loop: false,
    playback: true,    // Also play locally
  }
});
```

### Combined Audio/Video File

```typescript
// Use same URL for both - start video first
const mediaFile = {
  url: 'https://example.com/video.mp4',
  loop: true,
};

await stream.startVideo({ mediaFile });
await stream.startAudio({ mediaFile });
```

**Note**: Stopping video will also pause audio when using same file.

---

## Real-Time Media Streams

Access raw media streams for custom processing.

### Setup

```typescript
const rtms = client.getRealTimeMediaStreamsClient();
```

### Start RTMS

```typescript
await rtms.startRealTimeMediaStreams();
```

### Events

```typescript
client.on('real-time-media-streams-status-change', (payload) => {
  const { status } = payload;
  // Handle status changes
});
```

---

## Broadcast Streaming

Stream to external platforms.

### Setup

```typescript
const broadcast = client.getBroadcastStreamingClient();
```

### Events

```typescript
client.on('broadcast-streaming-status', (payload) => {
  const { status } = payload;
});
```

---

## Far-End Camera Control (PTZ)

Control other participants' PTZ cameras.

### Request Control

```typescript
await stream.requestFarEndCameraControl(userId);
```

### Respond to Request

```typescript
client.on('far-end-camera-request-control', async (payload) => {
  const { userId, displayName } = payload;

  if (confirm(`${displayName} wants to control your camera. Allow?`)) {
    await stream.approveFarEndCameraControl(userId);
  } else {
    await stream.declineFarEndCameraControl(userId);
  }
});
```

### Control Remote Camera

```typescript
// After approval
await stream.controlFarEndCamera(CameraControlCmd.ZoomIn, userId);
await stream.controlFarEndCamera(CameraControlCmd.ZoomOut, userId);
await stream.controlFarEndCamera(CameraControlCmd.Left, userId);
await stream.controlFarEndCamera(CameraControlCmd.Right, userId);
await stream.controlFarEndCamera(CameraControlCmd.Up, userId);
await stream.controlFarEndCamera(CameraControlCmd.Down, userId);

// Give up control
await stream.giveUpFarEndCameraControl(userId);
```

### Control Events

```typescript
client.on('far-end-camera-response-control', (payload) => {
  const { userId, approved, reason } = payload;
  // reason: FarEndCameraControlDeclinedReason
});

client.on('far-end-camera-in-control-change', (payload) => {
  const { oderId, isInControl } = payload;
});

client.on('far-end-camera-capability-change', (payload) => {
  const { userId, capability } = payload;
  // capability: { pan, tilt, zoom }
});
```

### Check Capability

```typescript
const capability = stream.getFarEndCameraPTZCapability(userId);
// { pan: boolean, tilt: boolean, zoom: boolean }
```
