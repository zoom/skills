# RTMS Lifecycle Flow

Complete flow from meeting/webinar/session start to media streaming.

## High-Level Flow

```
┌─────────────────────────────┐
│  Meeting/Webinar/Session    │
│  Starts                     │
└────────────┬────────────────┘
             │
             ▼
┌─────────────────────────────┐
│  Zoom sends webhook event   │
│  meeting.rtms_started  OR   │
│  webinar.rtms_started  OR   │
│  session.rtms_started       │
└────────────┬────────────────┘
         │
         ▼
┌──────────────────┐
│  Your server     │
│  receives        │
│  webhook         │
│                  │
│  RESPOND 200     │
│  IMMEDIATELY!    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Connect to      │
│  Signaling WS    │
│                  │
│  Send handshake  │
│  (msg_type: 1)   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Receive         │
│  handshake resp  │
│  (msg_type: 2)   │
│                  │
│  Extract media   │
│  server URL      │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Connect to      │
│  Media WS        │
│                  │
│  Send handshake  │
│  (msg_type: 3)   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Receive media   │
│  handshake resp  │
│  (msg_type: 4)   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Send Client     │
│  Ready to        │
│  Signaling       │
│  (msg_type: 7)   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Receive media   │
│  data:           │
│  - Audio (14)    │
│  - Video (15)    │
│  - Share (16)    │
│  - Transcript(17)│
│  - Chat (18)     │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Respond to      │
│  heartbeats      │
│  (12 -> 13)      │
└────────┬─────────┘
         │
         ▼
┌─────────────────────────────┐
│  Optional control-plane     │
│  actions during stream      │
│  - EVENT_SUBSCRIPTION       │
│  - VIDEO_SUBSCRIPTION_REQ   │
│  - STREAM_CLOSE_REQ         │
└────────────┬────────────────┘
             │
             ▼
┌─────────────────────────────┐
│  meeting/webinar/session    │
│  .rtms_stopped              │
│                             │
│  Close sockets              │
│  Cleanup                    │
└─────────────────────────────┘
```

## Detailed Steps

### Step 1: Receive Webhook

When RTMS starts, Zoom sends a webhook. The event name and payload differ by product:

**Meeting RTMS:**
```json
{
  "event": "meeting.rtms_started",
  "payload": {
    "account_id": "abc123",
    "object": {
      "meeting_id": "123456789",
      "meeting_uuid": "AbC123...",
      "host_id": "user123",
      "rtms_stream_id": "stream123==",
      "server_urls": "wss://rtms-sjc1.zoom.us/...",
      "signature": "pre_computed_signature"
    }
  }
}
```

**Webinar RTMS:**
```json
{
  "event": "webinar.rtms_started",
  "payload": {
    "account_id": "abc123",
    "object": {
      "meeting_id": "123456789",
      "meeting_uuid": "AbC123...",
      "host_id": "user123",
      "rtms_stream_id": "stream123==",
      "server_urls": "wss://rtms-sjc1.zoom.us/...",
      "signature": "pre_computed_signature"
    }
  }
}
```

> **Note**: Webinar payloads use `meeting_uuid`, NOT `webinar_uuid`.

**Video SDK RTMS:**
```json
{
  "event": "session.rtms_started",
  "payload": {
    "account_id": "abc123",
    "object": {
      "session_id": "SessionABC...",
      "rtms_stream_id": "stream123==",
      "server_urls": "wss://rtms-sjc1.zoom.us/...",
      "signature": "pre_computed_signature"
    }
  }
}
```

> **Note**: Video SDK payloads use `session_id` instead of `meeting_uuid`.

### Product Differences

| Aspect | Meetings | Webinars | Video SDK |
|--------|----------|----------|-----------|
| Webhook event | `meeting.rtms_started` | `webinar.rtms_started` | `session.rtms_started` |
| Payload ID field | `meeting_uuid` | `meeting_uuid` (same!) | `session_id` |
| App type | General App (OAuth) | General App (OAuth) | Video SDK App (SDK Key/Secret) |
| Participants | All participants | Panelists have full streams; attendees may not | All participants |
| Protocol after connect | Identical | Identical | Identical |

**CRITICAL**: Respond with HTTP 200 **IMMEDIATELY** before any processing!

```javascript
const RTMS_EVENTS = ['meeting.rtms_started', 'webinar.rtms_started', 'session.rtms_started'];

app.post('/webhook', (req, res) => {
  res.status(200).send();  // FIRST!
  
  const { event, payload } = req.body;
  if (RTMS_EVENTS.includes(event)) {
    handleRTMSStarted(payload);
  }
});
```

**Why?** If you delay, Zoom retries the webhook. The retry creates a second connection, which kicks out your first connection.

### Step 2: Connect to Signaling WebSocket

```javascript
const signalingWs = new WebSocket(payload.server_urls);

// Use meeting_uuid for meetings/webinars, session_id for Video SDK
const idValue = payload.meeting_uuid || payload.session_id;

signalingWs.on('open', () => {
  const signature = generateSignature(
    CLIENT_ID, 
    idValue, 
    payload.rtms_stream_id, 
    CLIENT_SECRET
  );

  signalingWs.send(JSON.stringify({
    msg_type: 1,                    // Handshake request
    protocol_version: 1,
    meeting_uuid: idValue,
    rtms_stream_id: payload.rtms_stream_id,
    signature: signature,
    buffer_data: false,             // Use false for live streaming; true preserves startup audio
    media_type: 9                   // Audio(1) + Transcript(8)
  }));
});
```

### Audio Buffer Control

The signaling handshake supports optional `buffer_data` for startup audio:

- `buffer_data: true` or omitted: RTMS buffers and later delivers up to 60 seconds of audio captured after the `rtms_started` webhook while signaling and media sockets are being established.
- `buffer_data: false`: RTMS drops that initial buffered audio and starts from live audio once the media socket is ready.
- This only controls the startup buffer before the signaling connection is established. It does not disable buffering caused by later signaling or media socket interruptions.

Use `true` for recording, compliance, and "do not miss the opening sentence" workflows. Use `false` for live streaming, low-latency live AI, or any pipeline where receiving a startup backlog is worse than missing the first few seconds.

### Step 3: Handle Signaling Response

```javascript
signalingWs.on('message', (data) => {
  const msg = JSON.parse(data);
  
  switch (msg.msg_type) {
    case 2:  // Handshake response
      if (msg.status_code === 0) {
        // Extract media server URL
        const mediaUrl = msg.media_server.server_urls.all;
        connectToMediaServer(mediaUrl);
      } else {
        console.error('Handshake failed:', msg.status_code);
      }
      break;
      
    case 12:  // Keep alive request
      signalingWs.send(JSON.stringify({
        msg_type: 13,
        timestamp: msg.timestamp
      }));
      break;
  }
});
```

### Step 4: Connect to Media WebSocket

```javascript
function connectToMediaServer(mediaUrl) {
  const mediaWs = new WebSocket(mediaUrl);
  
  mediaWs.on('open', () => {
    mediaWs.send(JSON.stringify({
      msg_type: 3,                  // Media handshake request
      protocol_version: 1,
      meeting_uuid: idValue,        // meeting_uuid or session_id
      rtms_stream_id: streamId,
      signature: signature,
      media_type: 9,                // Audio + Transcript
      payload_encryption: false,
      media_params: {
        audio: {
          content_type: 2,          // RAW_AUDIO
          sample_rate: 1,           // 16kHz
          channel: 1,               // Mono
          codec: 1,                 // L16 (PCM)
          data_opt: 1,              // Mixed stream
          send_rate: 20             // 20ms chunks
        },
        transcript: {
          content_type: 5,          // TEXT
          src_language: 9,          // English
          enable_lid: false         // Fixed language, no auto-switch
        }
      }
    }));
  });
}
```

### Step 5: Start Streaming

After media handshake succeeds, tell signaling you're ready:

```javascript
mediaWs.on('message', (data) => {
  const msg = JSON.parse(data);
  
  if (msg.msg_type === 4 && msg.status_code === 0) {
    // Media handshake success - tell signaling we're ready
    signalingWs.send(JSON.stringify({
      msg_type: 7,                  // Client ready
      rtms_stream_id: streamId
    }));
  }
});
```

### Step 6: Receive Media Data

```javascript
mediaWs.on('message', (data) => {
  const msg = JSON.parse(data);
  
  switch (msg.msg_type) {
    case 14:  // Audio
      const audioContent = normalizeBinaryContent(msg.content);
      processAudio(audioContent.buffer, audioContent.userName || msg.user_name, audioContent.timestamp || msg.timestamp, audioContent.length);
      break;
      
    case 15:  // Video
      const videoContent = normalizeBinaryContent(msg.content);
      processVideo(videoContent.buffer, videoContent.userName || msg.user_name, videoContent.timestamp || msg.timestamp, videoContent.length);
      break;
      
    case 16:  // Screen share
      const shareContent = normalizeBinaryContent(msg.content);
      processScreenShare(shareContent.buffer, shareContent.userName || msg.user_name, shareContent.timestamp || msg.timestamp, shareContent.length);
      break;
      
    case 17:  // Transcript
      console.log(`${getContentUserName(msg.content, msg.user_name)}: ${getContentData(msg.content)}`);
      break;
      
    case 18:  // Chat
      console.log(`[Chat] ${getContentUserName(msg.content, msg.user_name)}: ${getContentData(msg.content)}`);
      break;
      
    case 12:  // Keep alive
      mediaWs.send(JSON.stringify({
        msg_type: 13,
        timestamp: msg.timestamp
      }));
      break;
  }
});

function normalizeBinaryContent(content) {
  const data = getContentData(content);
  const buffer = Buffer.from(data, 'base64');

  return {
    buffer,
    userName: getContentUserName(content),
    timestamp: content && typeof content === 'object' ? content.timestamp : undefined,
    // Newer audio/video/screen-share payloads include original byte length.
    length: content && typeof content === 'object' && content.length ? content.length : buffer.length
  };
}

function getContentData(content) {
  return typeof content === 'string' ? content : content?.data;
}

function getContentUserName(content, fallback) {
  return content && typeof content === 'object' && content.user_name ? content.user_name : fallback;
}
```

### Step 6A: Track Available Participant Video Streams

When using the new single-individual-video mode, the signaling socket tells you whose camera is currently available.

```javascript
const activeVideoUsers = new Set();

function handleEventUpdate(msg) {
  const eventType = msg.event?.event_type;
  const participants = msg.event?.participants || [];

  if (eventType === 8) { // PARTICIPANT_VIDEO_ON
    for (const participant of participants) activeVideoUsers.add(participant.user_id);
  }

  if (eventType === 9) { // PARTICIPANT_VIDEO_OFF
    for (const participant of participants) activeVideoUsers.delete(participant.user_id);
  }
}
```

Use these events as the control-plane signal for which participant video streams are currently subscribable.

### Step 6B: Select One Participant Video Stream

```javascript
function subscribeToParticipantVideo(streamId, userId) {
  const signalingWs = signalingConnections.get(streamId);
  if (!signalingWs) return;

  signalingWs.send(JSON.stringify({
    msg_type: 28, // VIDEO_SUBSCRIPTION_REQ
    user_id: userId,
    subscribe: true,
    timestamp: Date.now()
  }));
}
```

Important constraint:

- only one participant stream can be active at a time
- the newest successful subscription replaces the previous selection

### Step 7: Handle Session End

```javascript
const RTMS_STOP_EVENTS = ['meeting.rtms_stopped', 'webinar.rtms_stopped', 'session.rtms_stopped'];

// Via webhook
app.post('/webhook', (req, res) => {
  res.status(200).send();
  
  const { event, payload } = req.body;
  
  if (RTMS_STOP_EVENTS.includes(event)) {
    const streamId = payload.rtms_stream_id;
    
    // Close connections
    signalingConnections.get(streamId)?.close();
    mediaConnections.get(streamId)?.close();
    
    // Cleanup
    signalingConnections.delete(streamId);
    mediaConnections.delete(streamId);
  }
});

// Also handle WebSocket close events
signalingWs.on('close', (code, reason) => {
  console.log('Signaling closed:', code, reason);
  // Implement reconnection if needed
});
```

### Optional: Client-Initiated Graceful Close

The backend can now ask RTMS to terminate the stream cleanly:

```javascript
function closeStream(streamId) {
  const signalingWs = signalingConnections.get(streamId);
  if (!signalingWs) return;

  signalingWs.send(JSON.stringify({
    msg_type: 21, // STREAM_CLOSE_REQ
    rtms_stream_id: streamId
  }));
}
```

Expect a `STREAM_CLOSE_RESP` followed by normal socket teardown.

## Session Tracking

**CRITICAL**: Track active sessions to prevent duplicate connections!

```javascript
const activeSessions = new Map();

function handleRTMSStarted(payload) {
  const streamId = payload.rtms_stream_id;
  
  // Check for existing connection
  if (activeSessions.has(streamId)) {
    console.log('Already connected to this stream, ignoring');
    return;
  }
  
  // Mark as active (meeting_uuid for meetings/webinars, session_id for Video SDK)
  activeSessions.set(streamId, {
    startTime: Date.now(),
    idValue: payload.meeting_uuid || payload.session_id
  });
  
  // Connect
  connectToRTMS(payload);
}

function handleRTMSStopped(payload) {
  const streamId = payload.rtms_stream_id;
  activeSessions.delete(streamId);
  // ... cleanup
}
```

## Error Handling

```javascript
// SDK state management (from Arlo sample)
try {
  client.join(payload);
} catch (error) {
  if (error.message?.includes('Invalid status')) {
    console.warn('SDK in invalid state, waiting to retry...');
    
    setTimeout(() => {
      handleRTMSStarted(payload);
    }, 2000);
  }
}
```

## Next Steps

- **[SDK Quickstart](../examples/sdk-quickstart.md)** - SDK handles all this automatically
- **[Manual WebSocket](../examples/manual-websocket.md)** - Full implementation code
- **[Common Issues](../troubleshooting/common-issues.md)** - Debugging connection problems
