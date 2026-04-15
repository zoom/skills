# Advanced Features Guide

## Preview (Pre-Session Device Testing)

Test audio/video devices before joining a session.

### Camera Preview

```typescript
import ZoomVideo from '@zoom/videosdk';

// Create local video track
const localVideoTrack = ZoomVideo.createLocalVideoTrack(deviceId);

// Start preview in element
await localVideoTrack.start(document.getElementById('preview-video'));

// Switch camera
await localVideoTrack.switchCamera(newCameraId);

// With virtual background
await localVideoTrack.start(element, { imageUrl: 'blur' });
await localVideoTrack.start(element, { imageUrl: 'https://example.com/bg.jpg' });

// Stop preview
localVideoTrack.stop();
```

### Mobile Camera Preview

```typescript
// Use facingMode instead of deviceId
const localVideoTrack = ZoomVideo.createLocalVideoTrack();

// Front camera (default)
await localVideoTrack.start(element, { facingMode: 'user' });

// Back camera
await localVideoTrack.start(element, { facingMode: 'environment' });
```

### Microphone Preview

```typescript
const localAudioTrack = ZoomVideo.createLocalAudioTrack(deviceId);

// Start microphone (enable input)
localAudioTrack.unmute();

// Get volume level for visualization
const volume = localAudioTrack.getCurrentVolume();
// Update volume meter UI

// Test microphone (records and plays back)
await localAudioTrack.testMicrophone({
  speakerId: speakerDeviceId,
  recordDuration: 5000,  // 5 seconds, max 10
});

// Stop preview
localAudioTrack.mute();
localAudioTrack.stop();
```

### Speaker Preview

```typescript
// Test speaker with audio playback
await localAudioTrack.testSpeaker({
  speakerId: deviceId,
  onAnalyseFrequency: (frequencyData) => {
    // Update speaker visualization UI
  }
});

// Stop speaker test
localAudioTrack.stopTestSpeaker();
```

### Device Change Detection

```typescript
navigator.mediaDevices.ondevicechange = async () => {
  const devices = await ZoomVideo.getDevices();
  // Update device selection UI
};
```

### In-Session Preview

Preview camera/virtual background without publishing:

```typescript
// Preview virtual background privately
await stream.previewVirtualBackground(element, 'blur', false, cameraId);
await stream.previewVirtualBackground(element, 'https://example.com/bg.jpg');

// Mirror self-view
stream.mirrorVideo(true);

// Stop preview
stream.stopPreviewVirtualBackground();
```

---

## Cloud Recording

**Note**: Only cloud recording is supported (not local recording).

### Setup

```typescript
const recording = client.getRecordingClient();
```

### Control Recording

```typescript
// Check permission
const canRecord = recording.canStartRecording();

// Start recording
await recording.startCloudRecording();

// Pause/Resume
await recording.pauseCloudRecording();
await recording.resumeCloudRecording();

// Stop recording
await recording.stopCloudRecording();

// Get status
const status = recording.getCloudRecordingStatus();
// 'Recording' | 'Paused' | 'Stopped'
```

### Recording Events

```typescript
client.on('recording-change', (payload) => {
  const { status } = payload;
  // status: 'Recording' | 'Paused' | 'Stopped'
});
```

### Recording Options (JWT)

Configure via JWT payload:

```javascript
{
  cloud_recording_option: 0,  // 0=manual, 1=auto-start
  cloud_recording_election: 0, // Recording type
  cloud_recording_transcript_option: 1, // Enable transcript
}
```

### Recording Content

- **Video**: Active speaker, gallery view, screen share
- **Audio**: Combined or per-user files
- **Chat**: Session messages
- **Resolution**: Up to 1080p

### Spotlight for Recording

```typescript
// Keep user as active speaker in recording
await stream.spotlightVideo(userId, true);
await stream.spotlightVideo(userId, false);
```

---

## Subsessions (Breakout Rooms)

Up to **50 subsessions** per session.

### Setup

```typescript
const subsession = client.getSubsessionClient();
```

### Create Subsessions

```typescript
// Create with names
const rooms = subsession.createSubsessions(['Room A', 'Room B', 'Room C']);

// Create with count (auto-named)
const rooms = subsession.createSubsessions(5);

// Create with distribution mode
// 1 = distribute users evenly
// 2 = manual assignment (default)
const rooms = subsession.createSubsessions(['Room A', 'Room B'], 1);
```

### Assign Users

```typescript
// Before opening - pre-assign
await subsession.openSubsessions({
  subsessionList: [
    { subsessionId: 'room1', subsessionName: 'Room A', userList: [userId1, userId2] },
    { subsessionId: 'room2', subsessionName: 'Room B', userList: [userId3, userId4] },
  ]
});

// After opening - dynamic assignment
await subsession.assignUserToSubsession(userId, subsessionId);

// Move user between rooms (host only)
await subsession.moveUserToSubsession(userId, newSubsessionId);
```

### Open Subsessions

```typescript
await subsession.openSubsessions({
  isTimerEnabled: true,
  timerDuration: 1800,        // 30 minutes in seconds
  isTimerAutoEnabled: true,   // Auto-close when timer ends
  allowParticipantsToReturnToMain: true,
});
```

### Manage Subsessions

```typescript
// List subsessions (host sees all, user sees own)
const rooms = subsession.getSubsessionList();

// Get unassigned users
const unassigned = subsession.getUnassignedUserList();

// Broadcast to all rooms (host only)
await subsession.broadcast('Meeting ends in 5 minutes');

// Close all subsessions
await subsession.closeAllSubsessions();
```

### User Navigation

```typescript
// Join subsession
await subsession.joinSubsession(subsessionId);

// Leave subsession (return to main)
await subsession.leaveSubsession();

// Request help from host
await subsession.askForHelp();
```

### Host Help Responses

```typescript
// Host receives help request
client.on('subsession-ask-for-help', (payload) => {
  const { oderId, displayName, subsessionId } = payload;
  // Show notification to host
});

// Host responds
await subsession.admitUserFromWaitingRoom(userId); // Join their room
await subsession.postponeHelping(userId);          // Later
```

### Subsession Events

```typescript
// Invitation to join
client.on('subsession-invite-to-join', (payload) => {
  const { subsessionId, subsessionName } = payload;
});

// Countdown timer
client.on('subsession-countdown', (payload) => {
  const { remainingTime } = payload;
});

// Broadcast message
client.on('subsession-broadcast-message', (payload) => {
  const { message } = payload;
});

// State change
client.on('subsession-state-change', (payload) => {
  const { status } = payload;
  // SubsessionStatus: NotStarted | InProgress | Closing | Closed
});

// Help response
client.on('subsession-ask-for-help-response', (payload) => {
  // type: 'Received' | 'Busy' | 'Ignore' | 'AlreadyInRoom'
});
```

---

## Live Streaming (RTMP)

Stream to unlimited viewers via YouTube, Facebook, Twitch, etc.

### Setup

```typescript
const liveStream = client.getLiveStreamClient();
```

### Start Stream

```typescript
// Get credentials from streaming platform
const streamUrl = 'rtmp://a.rtmp.youtube.com/live2';
const streamKey = 'your-stream-key';
const broadcastUrl = 'https://youtube.com/watch?v=xxx';

// Start streaming
await liveStream.startLiveStream(streamUrl, streamKey, broadcastUrl);
```

**Note**: Broadcast URL is required even if not needed by platform. Use `https://zoom.us` as fallback.

### Stop Stream

```typescript
await liveStream.stopLiveStream();
```

### Stream Events

```typescript
client.on('live-stream-status', (payload) => {
  const { status } = payload;
  // 'started' | 'stopped' | 'failed'
});
```

### Platform Setup

**YouTube**:
1. Enable live streaming on your account
2. Schedule a stream to get credentials
3. Cannot use YouTube Webcam service

**Facebook/Twitch**: Get stream URL and key from platform dashboard.

---

## Command Channel

Real-time data transmission for custom features.

### Specifications

- **Max message size**: 512 characters
- **Rate limit**: 35 commands/second
- **Use cases**: Reactions, game state, custom signaling

### Setup

```typescript
const command = client.getCommandClient();
```

### Send Commands

```typescript
// Send to all participants
await command.send('{"type":"reaction","emoji":"👍"}');

// Send to specific user
await command.send('{"type":"private","data":"hello"}', userId);

// Stringify objects
const data = { action: 'move', x: 100, y: 200 };
await command.send(JSON.stringify(data));
```

### Receive Commands

```typescript
client.on('command-channel-message', (payload) => {
  const { senderName, senderId, text } = payload;

  try {
    const data = JSON.parse(text);
    handleCommand(data);
  } catch {
    console.log('Raw command:', text);
  }
});
```

### Connection Status

```typescript
client.on('command-channel-status', (payload) => {
  const { status } = payload;
  // 'connected' | 'disconnected'
});
```

---

## Live Transcription & Translation

**Requires separate license** - contact Zoom Sales.

### Setup

```typescript
const transcription = client.getLiveTranscriptionClient();
```

### Start Transcription

```typescript
await transcription.startLiveTranscription();

// Get supported languages
const status = transcription.getLiveTranscriptionStatus();
const languages = status.transcriptionLanguage;
```

### Configure Languages

```typescript
// Set speaking language
await transcription.setSpeakingLanguage('en-US');

// Set translation target
await transcription.setTranslationLanguage('es');
```

### Receive Captions

```typescript
client.on('caption-message', (payload) => {
  const { displayName, text, done, language } = payload;

  if (done) {
    // Final caption for this utterance
    addCaption(`${displayName}: ${text}`);
  } else {
    // Interim caption (still speaking)
    updateCaption(`${displayName}: ${text}...`);
  }
});
```

### Host Controls

```typescript
// Disable captions for all (host only)
await transcription.disableCaptions();

// Check if captions enabled
const status = transcription.getLiveTranscriptionStatus();
```

### Transcription Events

```typescript
client.on('caption-status', (payload) => {
  // Transcription service status
});

client.on('caption-enable', (payload) => {
  // Caption enabled/disabled
});

client.on('caption-host-disable', (payload) => {
  // Host disabled captions
});
```

---

## PSTN Phone Dial-Out

### Prerequisites

- Video SDK Account
- Audio Conferencing Plan
- Active session

### Dial Out

```typescript
// Basic dial-out
await stream.inviteByPhone('+1', '2025550176', 'Jane Doe');

// With extension
await stream.inviteByPhone('+1', '2025550176-123', 'Jane Doe');

// With options
await stream.inviteByPhone('+1', '2025550176', 'Jane Doe', {
  callMe: true,       // Bind to current user's audio
  greeting: true,     // Play greeting
  pressingOne: true,  // Require pressing 1
});
```

### Get Supported Countries

```typescript
const countries = await stream.getSupportedCountries();
// [{ name: 'United States', code: '+1', id: 'US' }, ...]
```

### Monitor Call State

```typescript
client.on('dialout-state-change', (payload) => {
  const { state } = payload;

  switch (state) {
    case DialoutState.Calling:
      console.log('Calling...');
      break;
    case DialoutState.Ringing:
      console.log('Ringing...');
      break;
    case DialoutState.Accepted:
      console.log('Call accepted');
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
    case DialoutState.HangUp:
      console.log('Hung up');
      break;
  }
});
```

### Cancel / Hangup

```typescript
// Cancel pending call
await stream.cancelDialOut('+1', '2025550176');

// Hangup connected call
await stream.hangup();
```

---

## SIP/H.323 (CRC Devices)

Connect conference room devices via Cloud Room Connector.

### Dial Out to Device

```typescript
// Call SIP device
await stream.callCRCDevice('7357@test.plcm.vc', CRCProtocol.SIP);

// Call H.323 device
await stream.callCRCDevice('192.168.1.100', CRCProtocol.H323);
```

### Cancel Call

```typescript
await stream.cancelCallCRCDevice('7357@test.plcm.vc', CRCProtocol.SIP);
```

### Monitor State

```typescript
client.on('crc-call-out-state-change', (payload) => {
  const { code } = payload;

  switch (code) {
    case CRCReturnCode.Success:
      console.log('Connected');
      break;
    case CRCReturnCode.Ringing:
      console.log('Ringing...');
      break;
    case CRCReturnCode.Busy:
      console.log('Device busy');
      break;
    case CRCReturnCode.Fail:
      console.log('Call failed');
      break;
    case CRCReturnCode.Timeout:
      console.log('Timeout');
      break;
    case CRCReturnCode.Unreachable:
      console.log('Device unreachable');
      break;
  }
});
```

### Dial-In Access

```typescript
// Get SIP address for dial-in
const sipAddress = stream.getSessionSIPAddress();
// Format: {SESSION_NUMBER}.{PASSCODE}@global.zoomcrc.com
```

---

## Quality & Statistics

### Network Quality

```typescript
client.on('network-quality-change', (payload) => {
  const { userId, uplink, downlink } = payload;

  // Scores: 0-1 = poor, 2 = normal, 3-5 = good
  if (uplink <= 1 || downlink <= 1) {
    showNetworkWarning(userId);
  }
});
```

**Note**: Requires 2+ users with video enabled.

### Audio Statistics

```typescript
// Subscribe
stream.subscribeAudioStatisticData();

client.on('audio-statistic-data-change', (payload) => {
  const { encoding, decoding } = payload.data;

  // Encoding (sending)
  console.log('Sample Rate:', encoding.sample_rate);
  console.log('Bitrate:', encoding.bitrate, 'bps');
  console.log('Packet Loss:', encoding.avg_loss, '%');

  // Decoding (receiving)
  console.log('RTT:', decoding.rtt, 'ms');
  console.log('Jitter:', decoding.jitter, 'ms');
});

// Unsubscribe
stream.unsubscribeAudioStatisticData();
```

### Video Statistics

```typescript
// Subscribe (aggregate)
stream.subscribeVideoStatisticData();

client.on('video-statistic-data-change', (payload) => {
  const { encoding, decoding } = payload.data;

  console.log('Send FPS:', encoding.fps);
  console.log('Send Resolution:', `${encoding.width}x${encoding.height}`);
  console.log('Receive FPS:', decoding.fps);
});

// Per-user detailed stats (higher CPU)
stream.subscribeVideoStatisticData({ detailed: true });

client.on('video-detailed-data-change', (payload) => {
  // Per-user statistics
});

// Unsubscribe
stream.unsubscribeVideoStatisticData();
```

### Screen Share Statistics

```typescript
stream.subscribeShareStatisticData();

client.on('share-statistic-data-change', (payload) => {
  const { encoding, decoding } = payload.data;
  // Similar to video stats
});

stream.unsubscribeShareStatisticData();
```

### Max Renderable Videos

```typescript
// Check device capability
const maxVideos = stream.getMaxRenderableVideos();
console.log('Can render up to', maxVideos, 'videos');
```

### User Feedback

```typescript
// Post-session rating (1-5)
clientSideTelemetry.reportRating(4, 'Great video quality');
```

### System Resource Monitoring

```typescript
client.on('system-resource-usage-change', (payload) => {
  const { level } = payload;

  // SystemCPUPressureLevel
  // Nominal (0): < 30% CPU
  // Fair (1): 30-60% CPU
  // Serious (2): 60-90% CPU
  // Critical (3): > 90% CPU

  if (level >= 2) {
    // Reduce quality or disable features
    reduceVideoQuality();
  }
});
```
