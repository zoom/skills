# Error Codes Reference

## Error Handling

All async operations return `ExecutedResult` which is a Promise that resolves to `''` on success or rejects with `ExecutedFailure`.

```typescript
interface ExecutedFailure {
  type: ErrorTypes;
  reason: string;
  errorCode: number;
}

type ErrorTypes =
  | 'INVALID_OPERATION'
  | 'INTERNAL_ERROR'
  | 'OPERATION_TIMEOUT'
  | 'INSUFFICIENT_PRIVILEGES'
  | 'IMPROPER_MEETING_STATE'
  | 'INVALID_PARAMETERS'
  | 'OPRATION_LOCKED';
```

### Example Error Handling

```typescript
try {
  await client.join(topic, token, userName);
} catch (error) {
  const { type, reason, errorCode } = error as ExecutedFailure;

  switch (type) {
    case 'INVALID_OPERATION':
      console.error('Duplicate operation');
      break;
    case 'INTERNAL_ERROR':
      console.error('Server error - try again');
      break;
    case 'INSUFFICIENT_PRIVILEGES':
      console.error('Need host/manager role');
      break;
    case 'OPERATION_TIMEOUT':
      console.error('Timeout - retry');
      break;
    case 'IMPROPER_MEETING_STATE':
      // reason: 'closed' | 'on hold' | 'reconnecting'
      console.error('Invalid meeting state:', reason);
      break;
    case 'INVALID_PARAMETERS':
      console.error('Invalid parameters:', reason);
      break;
    case 'OPRATION_LOCKED':
      console.error('Feature locked:', reason);
      break;
  }
}
```

## Common/Internal Errors (1-2)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 1 | OPERATION_TIMEOUT | Service delay | Retry operation |
| 2 | INTERNAL_ERROR | Zoom service error | Contact Zoom support |

## Session Errors (200, 3000s, 4000s)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 200 | SESSION_FETCH_INFO_ERROR | Usually incorrect JWT field | Check JWT token fields |
| 3004 | SESSION_INCORRECT_PASSCODE | Wrong password | Verify password |
| 3009 | SESSION_USER_REMOVED | User blocked from rejoining | Contact host |
| 3010 | SESSION_ROLE_TYPE_ERROR | Invalid JWT role | Set role to 0 or 1 |
| 4004 | SESSION_ENDED | Meeting concluded | Session no longer exists |

## Client Errors (5000s)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 5000 | CLIENT_PLATFORM_UNSUPPORTED | No WebRTC support | Use supported browser |
| 5002 | CLIENT_SESSION_STATE_CLOSED | Called outside session | Join session first |
| 5012 | CLIENT_DUPLICATED_JOIN | Duplicate join attempt | Wait for first join |
| 5013 | CLIENT_INVALID_JOIN_PARAMETER | Malformed join request | Check parameters |

## Audio Errors (6010-6029)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 6010 | AUDIO_CAPTURE_FAILED | Permission denied or device unavailable | Request permission, check device |
| 6011 | AUDIO_CAPTURE_LOADING | Audio system initializing | Wait and retry |
| 6015 | AUDIO_NOT_STARTED | Mute/unmute without active audio | Call startAudio() first |
| 6020 | AUDIO_USER_FORBIDDEN_MICROPHONE | Microphone permission denied | Request permission |
| 6021 | AUDIO_NO_MICROPHONE | No microphone found | Check device connection |

## Video Errors (6100-6154)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 6100 | VIDEO_CAMERA_PERMISSION_DENIED | Camera permission denied | Request permission |
| 6101 | VIDEO_NO_CAMERA | No camera detected | Check device connection |
| 6103 | VIDEO_CAMERA_TAKEN | Camera in use | Close other applications |
| 6107 | VIDEO_VB_UNSUPPORTED | Virtual background unavailable | Check browser/SAB support |
| 6110 | VIDEO_NOT_STARTED | Video operation without started video | Call startVideo() first |

## Screen Share Errors (6200-6226)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 6200 | SCREEN_SHARE_USER_DENIED | User rejected share | User cancelled share picker |
| 6201 | SCREEN_SHARE_SYSTEM_DENIED | System permission denied | Enable in OS settings |
| 6204 | SCREEN_SHARE_HOST_PRIVILEGE_REQUIRED | Non-host tried to share | Unlock sharing or be host |
| 6210 | SCREEN_SHARE_IN_PROGRESS | Already sharing | Stop current share first |

## Chat Errors (7000-7011)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 7000 | CHAT_EMPTY_MESSAGE | Empty message | Provide message content |
| 7005 | CHAT_PRIVILEGE_LIMITED | No chat privilege | Check chat privilege settings |
| 7007 | CHAT_MISMATCH_FILE_TYPE | Invalid file type | Use allowed file types |
| 7008 | CHAT_EXCEED_FILE_SIZE | File too large | Reduce file size |
| 7011 | CHAT_RECEIVER_NOT_FOUND | Recipient not in session | Check user ID |

## Command Channel Errors (7100-7104)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 7100 | COMMAND_CHANNEL_NOT_CONNECTED | Channel not connected | Wait for connection |
| 7101 | COMMAND_CHANNEL_MESSAGE_TOO_LONG | Message exceeds limit | Reduce message size |
| 7104 | COMMAND_CHANNEL_RECEIVER_NOT_FOUND | Recipient not found | Check user ID |

## Recording Errors (7200-7203)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 7200 | RECORDING_DISABLED | Recording not enabled | Enable in account settings |
| 7201 | RECORDING_IN_PROGRESS | Already recording | Stop current recording |
| 7202 | RECORDING_PRIVILEGE_LIMITED | No recording privilege | Need host role |
| 7203 | RECORDING_NOT_IN_PROGRESS | Stop called without active recording | Start recording first |

## Transcription Errors (7300-7305)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 7300 | TRANSCRIPTION_DISABLED | Transcription not enabled | Enable in account settings |
| 7301 | TRANSCRIPTION_LANGUAGE_UNSUPPORTED | Language not supported | Use supported language |
| 7302 | TRANSCRIPTION_IN_PROGRESS | Already transcribing | Stop current transcription |

## Live Stream Errors (7400-7402)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 7400 | LIVESTREAM_NOT_STARTED | Stream not started | Start stream first |
| 7401 | LIVESTREAM_IN_PROGRESS | Already streaming | Stop current stream |
| 7402 | LIVESTREAM_INVALID_URL | Invalid RTMP URL | Check URL format |

## Subsession Errors (7500-7510)

| Code | Name | Description | Solution |
|------|------|-------------|----------|
| 7500 | SUBSESSION_NOT_SUPPORTED | Feature not enabled | Enable in account |
| 7501 | SUBSESSION_NOT_STARTED | Subsessions not open | Open subsessions first |
| 7502 | SUBSESSION_PRIVILEGE_LIMITED | No subsession privilege | Need host role |
| 7505 | SUBSESSION_NOT_IN_SUBSESSION | User not in subsession | Join subsession first |

## Active Media Failure Codes

Handle via `active-media-failed` event:

```typescript
client.on('active-media-failed', (payload) => {
  switch (payload.code) {
    // Audio failures
    case 101: // AudioConnectionFailed
      console.error('Audio connection failed');
      break;
    case 102: // AudioStreamEnded
      console.error('Audio stream ended');
      break;
    case 103: // MicrophonePermissionReset
      console.error('Microphone permission revoked');
      break;
    case 104: // AudioStreamFailed
      console.error('Failed to get audio data');
      break;
    case 105: // MicrophoneMuted
      console.error('Microphone muted in system');
      break;
    case 106: // AudioStreamMuted
      console.error('Sent audio interrupted');
      break;
    case 107: // AudioPlaybackInterrupted
      console.error('Remote audio playback interrupted');
      break;

    // Video failures
    case 201: // VideoConnectionFailed
      console.error('Video connection failed');
      break;
    case 202: // VideoStreamEnded
      console.error('Video stream ended');
      break;
    case 203: // CameraPermissionReset
      console.error('Camera permission revoked');
      break;
    case 204: // WebGlContextInvalid
      console.error('WebGL context lost');
      break;
    case 205: // WasmOutOfMemory
      console.error('Out of memory');
      break;
    case 206: // VideoStreamFailed
      console.error('Failed to get video data');
      break;
    case 207: // VideoStreamMuted
      console.error('Video stream interrupted');
      break;

    // Sharing failures
    case 301: // SharingStreamFailed
      console.error('Failed to get sharing data');
      break;
  }
});
```

## Debugging Tips

1. **Check JWT first** for session errors (200, 3xxx)
2. **Check permissions** for media errors (6xxx)
3. **Check role** for privilege errors (INSUFFICIENT_PRIVILEGES)
4. **Check account settings** for feature errors (recording, transcription)
5. **Enable logging** with `client.getLoggerClient()`

```typescript
const logger = client.getLoggerClient({
  debugMode: true,
  trackingId: 'my-tracking-id',
});
```
