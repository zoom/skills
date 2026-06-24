# Plugin SDK Capabilities

Use this grouped inventory when deciding whether Zoom Plugin SDK can control an installed Zoom Workplace desktop client for a native companion app.

Availability depends on Zoom Workplace version, account settings, meeting type, webinar type, role, host policy, and platform package version. Verify exact signatures in the platform child skill before implementation.

If the request could also mean Meeting SDK for Windows or macOS, read [FAQ](faq.md) before routing.

## Setup And Lifecycle

- Initialize Plugin SDK with OAuth tokens.
- Authenticate against installed Zoom Workplace.
- Connect through local IPC transport.
- Detect Plugin SDK authentication results.
- Detect IPC connection status changes.
- Uninitialize Plugin SDK during shutdown.

## Meeting Start And Navigation

- Start meetings through Zoom Workplace.
- Join meetings through Zoom Workplace.
- Open Zoom settings windows by tab.
- Navigate Zoom Workplace to supplied URLs.

## Meeting State And Metadata

- Get current meeting status details.
- Get current meeting properties from client.
- Set active meeting topic text.
- Leave meetings through Zoom Workplace.
- End meetings through Zoom Workplace.
- Detect meeting status transitions from client.

## Participants And Roles

- Get active participant lists from meetings.
- Get users by participant ID.
- Get local participant identity details.
- Get user roles by participant ID.
- Change participant display names during meetings.
- Assign users as meeting co-hosts.
- Revoke co-host role from users.
- Make another participant meeting host.
- Reclaim host role when permitted.
- Invite users by email address.

## Participant Permissions

- Allow participants to start video.
- Check participant video-start permission state.
- Allow participants to share content.
- Check participant content-sharing permission state.
- Allow participants to rename themselves.
- Check participant rename permission state.
- Allow participants to request recording.
- Check local-recording request permission state.
- Allow participants to share whiteboards.
- Check whiteboard-sharing permission state currently.
- Allow participants to use chat.
- Check participant chat permission state.
- Allow participants to unmute themselves.
- Check self-unmute permission state currently.

## Waiting Room

- Admit all waiting-room users together.
- Admit specific waiting-room users individually.
- Put participants into waiting rooms.
- Expel participants from waiting rooms.
- Check waiting-room feature support availability.
- Enable waiting-room feature controls.
- Check waiting-room enabled state currently.
- Get waiting-room user list details.

## Audio Controls

- Join computer audio in meetings.
- Leave computer audio in meetings.
- Mute local meeting audio input.
- Unmute local meeting audio input.
- Toggle local meeting audio state.
- Mute all participant audio streams.
- Unmute all participant audio streams.
- Enable mute-on-entry for joining participants.
- Check mute-on-entry configuration state currently.
- Detect local audio mute changes.

## Audio Settings

- Get microphone or speaker device list.
- Select microphone or speaker device.
- Enable high-fidelity music mode setting.
- Enable audio echo cancellation setting.
- Enable stereo audio mode setting.
- Check original sound support state.
- Enable original sound feature setting.
- Toggle original sound feature state.
- Detect original sound status changes.

## Video Controls

- Mute local camera video feed.
- Unmute local camera video feed.
- Toggle local camera video state.
- Pin local camera video feed.
- Unpin local camera video feed.
- Spotlight local camera video feed.
- Remove local camera video spotlight.
- Toggle local video spotlight state.
- Spotlight another participant video feed.
- Remove another participant video spotlight.
- Remove all active video spotlights.
- Show avatar instead of video.
- Hide avatar instead of video.
- Detect local video status changes.

## Video Settings

- Get available camera device list.
- Select active camera device directly.
- Switch to next camera source.
- Set virtual background configuration values.
- Enable camera mirror effect setting.
- Enable beauty face effect setting.
- Set beauty face intensity value.
- Enable original-size video feature setting.
- Enable HD video feature setting.
- Enable hidden self-view feature setting.

## Sharing

- Start sharing an application window.
- Start sharing a monitor display.
- Start sharing computer audio only.
- Start sharing a Zoom whiteboard.
- Start sharing a camera feed.
- Start sharing a video file.
- Start sharing application frame content.
- Stop active screen sharing session.
- Pause or resume active sharing.
- Check whether sharing can start.

## Share Settings And Events

- Enable shared computer sound setting.
- Get current audio sharing mode.
- Set current audio sharing mode.
- Enable optimized shared-video feature setting.
- Detect share options changes live.
- Detect outgoing share start events.
- Detect outgoing share stop events.
- Detect share pause status changes.
- Detect share content-type changes live.
- Detect share status changes live.

## Recording

- Get cloud recording status currently.
- Get local recording status currently.
- Start cloud recording when permitted.
- Stop cloud recording when permitted.
- Pause cloud recording when permitted.
- Resume cloud recording when permitted.
- Start local recording when permitted.
- Stop local recording when permitted.
- Pause local recording when permitted.
- Resume local recording when permitted.
- Request local recording privilege explicitly.
- Detect recording status changes live.

## Chat And Reactions

- Send meeting chat messages programmatically.
- Get unread chat count currently.
- Set participant chat privileges programmatically.
- Check whether chat window shows.
- Detect unread chat count changes.
- Detect chat window visibility changes.
- Send emoji reactions to meetings.
- Send dynamic emoji reactions to meetings.
- Send feedback reactions to meetings.
- Lower all raised hands programmatically.

## Captions And Transcription

- Start live transcription when supported.
- Enable closed caption feature setting.
- Check meeting closed-caption support state.

## Q&A And Webinar

- Get attendee lists for webinars.
- Check Q&A feature support state.
- Enable Q&A feature setting programmatically.
- Enable Q&A comment feature setting.
- Enable Q&A upvoting feature setting.
- Enable anonymous question submission setting.
- Enable attendee viewing all questions.
- Set panelist chat privileges programmatically.
- Get panelist chat privileges currently.
- Allow webinar attendees to talk.
- Check attendee talk permission state.
- Promote attendees to panelists directly.
- Demote panelists to attendees directly.

## Annotation

- Start annotation mode during sharing.
- Stop annotation mode during sharing.
- Clear selected annotation types programmatically.

## Zoom Workplace UI

- Switch meeting UI components programmatically.
- Switch meeting view types programmatically.
- Move wall view pages forward.
- Toggle meeting fullscreen mode programmatically.
- Get current meeting view type.
- Detect meeting view type changes.
- Detect fullscreen status changes live.
- Detect group layout updates live.

## Statistics

- Query audio performance statistics from client.
- Query video performance statistics from client.
- Query overall performance statistics from client.
- Query screen-sharing performance statistics from client.
