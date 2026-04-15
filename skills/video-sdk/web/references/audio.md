# Audio Guide

## Starting Audio

Audio requires a **user gesture** (click/tap) to start due to browser autoplay policies.

```typescript
const stream = client.getMediaStream();

// Basic start (connects microphone)
await stream.startAudio();

// With options
await stream.startAudio({
  mute: true,                      // Start muted
  speakerOnly: false,              // true = speaker only, no mic
  backgroundNoiseSuppression: true, // Reduce background noise
  microphoneId: 'device-id',       // Specific microphone
  speakerId: 'device-id',          // Specific speaker
  originalSound: false,            // High fidelity audio
  highBitrate: false,              // 128 kbps bitrate
});
```

### CRITICAL: Default Audio Behavior

**Important**: `startAudio()` joins audio in an **unmuted** state by default, even without the `mute` option. To ensure audio starts muted:

```typescript
// RECOMMENDED: Explicitly mute after starting
await stream.startAudio();
await stream.muteAudio();

// OR: Use the mute option (may not work reliably)
await stream.startAudio({ mute: true });
```

**Why this matters**: Users expect to join muted by default for privacy. Always explicitly call `muteAudio()` after `startAudio()` completes.

### Start Audio Options

| Option | Type | Description |
|--------|------|-------------|
| `mute` | boolean | Join with microphone muted |
| `speakerOnly` | boolean | Connect speaker only, no microphone |
| `backgroundNoiseSuppression` | boolean | Reduce background noise (requires SAB for WebAssembly) |
| `microphoneId` | string | Specific microphone device ID |
| `speakerId` | string | Specific speaker device ID |
| `originalSound` | boolean \| object | High fidelity audio `{ hifi?: boolean, stereo?: boolean }` |
| `highBitrate` | boolean | 128 kbps bitrate |
| `syncButtonsOnHeadset` | boolean | Sync mute state with compatible headsets |
| `autoStartAudioInSafari` | boolean | For auto-starting in Safari without gesture |
| `mediaFile` | MediaPlaybackFile | Use media file as audio input |

## Mute / Unmute

```typescript
// Mute self
await stream.muteAudio();

// Unmute self
await stream.unmuteAudio();

// Check mute status
const user = client.getCurrentUserInfo();
if (user.muted === true) {
  console.log('Muted');
} else if (user.muted === false) {
  console.log('Unmuted');
} else {
  console.log('Not connected to audio');
}
```

### Host Unmute Request

Hosts cannot directly unmute others. They can request unmute:

```typescript
// Host requests user to unmute
await stream.unmuteAudio(userId);

// User receives event
client.on('host-ask-unmute-audio', () => {
  // Show UI prompt to user
  const accept = confirm('Host is asking you to unmute. Accept?');
  if (accept) {
    stream.unmuteAudio();
  }
});
```

## Stop Audio

```typescript
// Completely disconnect from audio
stream.stopAudio();
```

**Note**: Use `muteAudio()` for temporary muting. Only use `stopAudio()` when intentionally leaving the audio stream.

## Device Management

### Microphones

```typescript
// Get available microphones
const mics = stream.getMicList();
// [{ label: 'Built-in Microphone', deviceId: '...' }, ...]

// Get active microphone
const activeMic = stream.getActiveMicrophone();

// Switch microphone
await stream.switchMicrophone('new-device-id');
```

### Speakers

```typescript
// Get available speakers
const speakers = stream.getSpeakerList();

// Get active speaker
const activeSpeaker = stream.getActiveSpeaker();

// Switch speaker
await stream.switchSpeaker('new-device-id');
```

### Device Change Event

```typescript
client.on('device-change', async () => {
  // Re-enumerate devices when connected/disconnected
  const mics = stream.getMicList();
  const speakers = stream.getSpeakerList();
  // Update device selection UI
});
```

## Audio Events

### Current Audio Change

```typescript
client.on('current-audio-change', (payload) => {
  switch (payload.action) {
    case 'join':
      console.log('Joined audio');
      break;
    case 'leave':
      console.log('Left audio, reason:', payload.source);
      // source: 'active' | 'failover' | 'audio stream is ended by system' | 'pstn' | 'microphone error'
      break;
    case 'muted':
      console.log('Muted, source:', payload.source);
      // source: 'active' | 'passive(mute one)' | 'passive(mute all)'
      break;
    case 'unmuted':
      console.log('Unmuted');
      break;
  }
});
```

### Active Speaker

```typescript
client.on('active-speaker', (payload) => {
  // Fires when someone speaks for > 1 second
  payload.forEach(speaker => {
    console.log(`Active speaker: userId=${speaker.oderId}`);
    // Highlight in UI
  });
});
```

### Audio Level

```typescript
client.on('current-audio-level-change', (payload) => {
  // Real-time audio level for visual indicators
  console.log('Audio level:', payload.level);
});
```

### Speaking While Muted

```typescript
client.on('speaking-while-muted', () => {
  // Show notification to user
  alert('You are speaking while muted!');
});
```

## Audio Statistics

```typescript
// Subscribe to audio statistics
stream.subscribeAudioStatisticData();

client.on('audio-statistic-data-change', (payload) => {
  const { encoding, decoding } = payload.data;

  // Encoding (sending) stats
  console.log('Send - Bitrate:', encoding.bitrate, 'bps');
  console.log('Send - Jitter:', encoding.jitter, 'ms');
  console.log('Send - Packet Loss:', encoding.avg_loss, '%');

  // Decoding (receiving) stats
  console.log('Receive - Bitrate:', decoding.bitrate, 'bps');
  console.log('Receive - RTT:', decoding.rtt, 'ms');
});

// Unsubscribe when done
stream.unsubscribeAudioStatisticData();
```

### AudioQosData Interface

```typescript
interface AudioQosData {
  sample_rate: number;  // Audio sample rate
  rtt: number;          // Round trip time
  jitter: number;       // Jitter in ms
  avg_loss: number;     // Average packet loss %
  max_loss: number;     // Maximum packet loss %
  bandwidth: number;    // Bandwidth in bps
  bitrate: number;      // Bitrate in bps
}
```

## Background Noise Suppression

Reduces background noise like dog barking, lawn mower, clapping, fans, pen tapping.

```typescript
// Enable at start
await stream.startAudio({
  backgroundNoiseSuppression: true,
});

// Or toggle during session
await stream.enableBackgroundNoiseSuppression(true);
await stream.enableBackgroundNoiseSuppression(false);
```

**Note**: Requires SharedArrayBuffer for WebAssembly audio mode.

## Original Sound (High Fidelity)

For music or professional audio:

```typescript
await stream.startAudio({
  originalSound: {
    hifi: true,    // High fidelity
    stereo: true,  // Stereo audio
  }
});
```

**Note**: `originalSound` and `backgroundNoiseSuppression` conflict. Enabling `originalSound` disables noise suppression.

## Phone Dial-Out (PSTN)

```typescript
// Get supported countries
const countries = await stream.getSupportedCountries();
// [{ name: 'United States', code: '+1', id: 'US' }, ...]

// Dial out
await stream.dialOut('+1', '1234567890', {
  callMe: true,       // Bind phone audio to current user
  greeting: true,     // Play greeting before connecting
  pressingOne: true,  // Require pressing 1 to connect
});

// Check dial state
client.on('dialout-state-change', (payload) => {
  switch (payload.state) {
    case DialoutState.Calling:
      console.log('Calling...');
      break;
    case DialoutState.Ringing:
      console.log('Ringing...');
      break;
    case DialoutState.Success:
      console.log('Connected!');
      break;
    case DialoutState.Busy:
      console.log('Line busy');
      break;
    case DialoutState.Fail:
      console.log('Call failed');
      break;
  }
});

// Cancel dial
await stream.cancelDialOut('+1', '1234567890');
```

## Auto-Play Handling

Some browsers block audio auto-play:

```typescript
client.on('auto-play-audio-failed', async () => {
  // Show UI prompt for user to click
  const button = document.getElementById('enable-audio');
  button.style.display = 'block';
  button.onclick = async () => {
    await stream.startAudio();
    button.style.display = 'none';
  };
});
```

## Error Codes

| Code | Name | Description |
|------|------|-------------|
| 6010 | AUDIO_CAPTURE_FAILED | Permission denied, device unavailable, or in-use |
| 6011 | AUDIO_CAPTURE_LOADING | Audio system initializing |
| 6015 | AUDIO_NOT_STARTED | Mute/unmute without active audio |
