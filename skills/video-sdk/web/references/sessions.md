# Sessions Guide

## Session Overview

- **Capacity**: Up to 1,000 concurrent users per session
- **Creation**: On-demand, no scheduling required
- **Limit**: One session per browser tab
- **Concurrent**: Unlimited sessions across tabs

## Session Lifecycle

### 1. Pre-Session Setup

```typescript
// Check browser compatibility FIRST
const { audio, video, screen } = ZoomVideo.checkSystemRequirements();
if (!audio || !video) {
  alert("Your browser does not support video conferencing");
  return;
}

// Preload assets for better performance
ZoomVideo.preloadDependentAssets();

// Check supported features
const features = ZoomVideo.checkFeatureRequirements();
console.log("Supported:", features.supportFeatures);
console.log("Unsupported:", features.unSupportFeatures);
```

### 2. Initialize & Join

```typescript
const client = ZoomVideo.createClient();

// Initialize with recommended options
await client.init("en-US", "Global", {
  patchJsMedia: true, // Auto-receive non-breaking enhancements
  stayAwake: true, // Prevent device sleep during session
  leaveOnPageUnload: true, // Clean disconnect on page close
});

// Join session
await client.join(
  "session-name", // Must match JWT 'tpc' field
  jwtToken, // Generated on server
  "Display Name", // Up to 200 characters
  "password123", // Optional, max 10 characters
  40 // Idle timeout in minutes (default: 40)
);

// Get media stream after joining
const stream = client.getMediaStream();
```

### 3. During Session

```typescript
// Get session info
const sessionInfo = client.getSessionInfo();
// { topic, password, userName, userId, isInMeeting, sessionId }

// Get current user
const me = client.getCurrentUserInfo();

// Get all participants
const participants = client.getAllUser();

// Get specific user
const user = client.getUser(userId);

// Check role
const isHost = client.isHost();
const isManager = client.isManager();
```

### 4. Leave Session

```typescript
// Participant leaves
await client.leave();

// Host ends session for everyone
await client.leave(true);

// Cleanup
await ZoomVideo.destroyClient();
```

## User Roles & Privileges

| Action               | Host | Manager | Participant |
| -------------------- | :--: | :-----: | :---------: |
| View all user info   |  ✅  |   ✅    |     ✅      |
| Mute others' audio   |  ✅  |   ✅    |     ❌      |
| Remove participants  |  ✅  |   ✅    |     ❌      |
| Change display names |  ✅  |   ✅    |     ❌      |
| Lock screen sharing  |  ✅  |   ✅    |     ❌      |
| Transfer host role   |  ✅  |   ❌    |     ❌      |
| Promote to manager   |  ✅  |   ❌    |     ❌      |
| Revoke manager       |  ✅  |   ❌    |     ❌      |
| Start livestream     |  ✅  |   ❌    |     ❌      |
| End session          |  ✅  |   ❌    |     ❌      |

### Role Management

```typescript
// Host transfers host role (there can only be one host)
await client.makeHost(userId);

// Host promotes user to manager (multiple managers allowed)
await client.makeManager(userId);

// Host revokes manager privileges
await client.revokeManager(userId);

// Original host reclaims host role
await client.reclaimHost();

// Remove participant from session
await client.removeUser(userId);

// Change display name
await client.changeName("New Name", userId); // userId optional for self
```

## JWT Token

The JWT token must be generated on your server using HMAC SHA256. Never expose SDK secret in client code.

### Required Claims

| Claim       | Description                                                         |
| ----------- | ------------------------------------------------------------------- |
| `app_key`   | Your Video SDK key                                                  |
| `tpc`       | Session name (max 200 chars), must match `join()` topic             |
| `role_type` | `1` = host/co-host, `0` = participant                               |
| `version`   | Always set to `1`                                                   |
| `iat`       | Token issued timestamp (Unix epoch)                                 |
| `exp`       | Expiration: minimum 1800s after `iat`, maximum 48 hours after `iat` |

### Optional Claims

| Claim                               | Description                                                         |
| ----------------------------------- | ------------------------------------------------------------------- |
| `user_key`                          | Unique user identifier (max 36 chars)                               |
| `session_key`                       | Session identifier for cloud recording (max 36 chars)               |
| `geo_regions`                       | Comma-separated data centers: `AU,BR,CA,DE,HK,IN,JP,CN,MX,NL,SG,US` |
| `cloud_recording_option`            | `0` = combined video, `1` = separate files per user                 |
| `cloud_recording_election`          | `1` = record user's self-view                                       |
| `cloud_recording_transcript_option` | `0` = none, `1` = transcript, `2` = transcript + summary            |
| `video_webrtc_mode`                 | `0` = disable, `1` = enable WebRTC video                            |
| `audio_webrtc_mode`                 | `1` = enable WebRTC audio (Web SDK only)                            |
| `telemetry_tracking_id`             | For Web SDK telemetry tracking                                      |

### Node.js Example (jsrsasign)

```javascript
import { KJUR } from "jsrsasign";

const iat = Math.floor(Date.now() / 1000) - 30; // Subtract 30s for clock skew
const exp = iat + 60 * 60 * 2; // 2 hours

const payload = {
  app_key: process.env.ZOOM_SDK_KEY,
  tpc: "session-name",
  role_type: 1,
  version: 1,
  iat,
  exp,
};

const token = KJUR.jws.JWS.sign(
  "HS256",
  JSON.stringify({ alg: "HS256", typ: "JWT" }),
  JSON.stringify(payload),
  process.env.ZOOM_SDK_SECRET
);
```

### Troubleshooting

- Ensure account is licensed for Video SDK
- Verify using correct SDK key and secret
- Check `tpc` matches session name exactly (max 200 chars)
- Session password max 10 characters if used
- Token expiration must be 1800s-48h from `iat`

### Role Behavior

- First user with `role_type: 1` becomes **host**
- Subsequent users with `role_type: 1` become **managers**
- Users with `role_type: 0` are **participants**

## Connection Events

```typescript
client.on("connection-change", (payload) => {
  switch (payload.state) {
    case "Connected":
      console.log("Successfully joined session");
      break;
    case "Reconnecting":
      console.log("Reconnecting...", payload.reason);
      // reason: 'failover' | 'join breakout room' | 'move to breakout room' | 'back to main session'
      break;
    case "Closed":
      console.log("Session closed");
      break;
    case "Fail":
      console.log("Connection failed:", payload.reason);
      break;
  }
});
```

## User Events

```typescript
// User joined
client.on("user-added", (participants: Participant[]) => {
  participants.forEach((user) => {
    console.log(`${user.displayName} joined`);
    // Add to UI
  });
});

// User updated (name change, audio/video state, etc.)
client.on("user-updated", (participants: Participant[]) => {
  participants.forEach((user) => {
    console.log(`${user.displayName} updated`);
    // Update UI
  });
});

// User left
client.on("user-removed", (participants: Participant[]) => {
  participants.forEach((user) => {
    console.log(`${user.displayName} left`);
    // Remove from UI
  });
});
```

## Participant Interface

```typescript
interface Participant {
  userId: number;
  displayName: string;
  audio: "" | "computer" | "phone"; // Audio connection type
  muted?: boolean; // undefined if not connected to audio
  isHost: boolean;
  isManager: boolean;
  avatar?: string; // From Zoom profile
  bVideoOn: boolean; // Video is on
  sharerOn: boolean; // Is sharing screen
  sharerPause: boolean; // Share is paused
  bVideoShare?: boolean; // Share optimized for video
  bShareAudioOn?: boolean; // Sharing tab audio
  isPhoneUser?: boolean; // Connected via phone
  userGuid?: string; // Unified ID across subsessions
  isAllowIndividualRecording: boolean;
  isVideoConnect: boolean; // Has camera connected
  userIdentity?: string; // From JWT payload
  isSpeakerOnly?: boolean; // Speaker only, no mic
  subsessionId?: string; // If in subsession
}
```

## Best Practices

1. **Always check compatibility** before init with `checkSystemRequirements()`
2. **Preload assets** early with `preloadDependentAssets()` for faster join
3. **Handle reconnection** gracefully in `connection-change` event
4. **Display user list** with real-time status indicators
5. **Clean up** with `destroyClient()` after leaving
6. **Generate JWT server-side** - never expose SDK secret
7. **Set `leaveOnPageUnload: true`** to handle browser close/refresh
