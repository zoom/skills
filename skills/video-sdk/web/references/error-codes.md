<!-- Source: https://developers.zoom.us/docs/video-sdk/web/error-codes.md -->
<!-- Fetched: 2026-07-17 -->

# Error codes and troubleshooting


See the following table for a list of common error codes, descriptions, and troubleshooting suggestions.

## Common or internal exception

This type of error is caused by a Zoom service issue. If you encounter this error, please report it to Zoom.

-   `1` - OPERATION_TIMEOUT
-   `2` - INTERNAL_ERROR

## Session exception

-   `200` - SESSION_FETCH_INFO_ERROR - Session creation or retrieval error. Usually caused by an incorrect field value in the JWT token.
    -   connection error
    -   The token has expired, is over 2 days old, or is ineffective
    -   Password max of 10 characters
    -   This account does not exist or does not belong to you
    -   Verify JWT failed
    -   invalid session key
-   `3004` - SESSION_INCORRECT_PASSCODE - The session passcode is incorrect.
-   `3009` - SESSION_USER_REMOVED - The user was removed by the session host and is prevented from rejoining the same session.
-   `3010` - SESSION_ROLE_TYPE_ERROR - Incorrect role type in the JWT token.
-   `4003` - SESSION_INVALID_PARAMETER - Incorrect parameters for connecting to the Zoom server. If you encounter this issue, please report it to Zoom.
-   `4004` - SESSION_ENDED - The session has ended.

## Client exception

-   `5000` - CLIENT_PLATFORM_UNSUPPORTED - Occurs in `client.init` because the platform does not meet the basic WebRTC requirements.
-   `5001` - CLIENT_DEPENDENT_ASSETS_INACCESSIBLE - Occurs in `client.init`, due to the inability to access self-deployed dependent assets.
-   `5002` - CLIENT_SESSION_STATE_CLOSED - Called a function that requires the user to be in a session, but the session is closed.
-   `5003` - CLIENT_SESSION_STATE_RECONNECTING - Called a function that requires the user to be in a session, but the session is reconnecting.
-   `5006` - CLIENT_HOST_OR_MANAGER_PERMISSION_REQUIRED - Operation requires host or manager permission.
-   `5007` - CLIENT_HOST_PERMISSION_REQUIRED - Operation requires host permission.
-   `5008` - CLIENT_ORIGIN_HOST_PERMISSION_REQUIRED - Operation requires origin host permission.
-   `5011` - CLIENT_MISMATCH_USER - User ID doesn't match.
-   `5012` - CLIENT_DUPLICATED_JOIN - Occurs in `client.join` due to a duplicate call while the user is already in a session.
-   `5013` - CLIENT_INVALID_JOIN_PARAMETER - Occurs in `client.join` due to an incorrect parameter.
    -   invalid signature
    -   Topic, token and userName must be string and required
    -   The maximum length of topic is 200
    -   The maximum length of username is 200
    -   The maximum length of password is 10
-   `5014` - CLIENT_HOST_DISALLOW_RENAME - Occurs in `client.changeName` due to the host disallowing users from changing their names themselves.

## Stream exception

-   `6000` - STREAM_MISMATCH_DEVICE - The device ID (microphone, audio speaker, camera) does not match any device in the list.
-   `6001` - STREAM_MISMATCH_USER - The User ID doesn't match.
-   `6002` - STREAM_SESSION_JOIN_REQUIRED - Occurs when a method is called before `client.join` has resolved.
-   `6003` - STREAM_MISMATCH_RENDER_ELEMENT - Occurs in video or sharing-related functions when the target element does not match the required render element type.

### Audio exception

-   `6010` - AUDIO_CAPTURE_FAILED Occurs in `stream.startAudio`, for the following reasons:
    -   `1` - Permission denied by user
    -   `2` - Permission denied by system
    -   `3` - Device in use
    -   `4` - No device found
    -   `5` - Unknown reason
    -   `6` - Constraint violation
    -   `7` - Permission dismissed by user
-   `6011` - AUDIO_CAPTURE_LOADING - Occurs in `stream.startAudio` or `stream.stopAudio` when the audio capture is loading.
-   `6012` - AUDIO_IN_PSTN - Occurs in `stream.startAudio` when the audio is already connected by a PSTN call.
-   `6013` - AUDIO_ENCODING_REQUIRED - Occurs in `stream.startAudio`, only on legacy Safari browsers when the audio decoding or encoding is required.
-   `6014` - AUDIO_SHARE_AUDIO_CONFLICT - Occurs in `stream.stopAudio`, only on legacy Chrome browsers when sharing tab audio or system audio.
-   `6015` - AUDIO_NOT_STARTED - Occurs in `stream.muteAudio` or `stream.unmuteAudio` or `stream.muteShareAudio` or `stream.unmuteShareAudio` when audio or shared audio has not been started.
-   `6017` - SECONDARY_AUDIO_UNSUPPORTED - Occurs in `stream.startSecondaryAudio` due to the platform being unsupported.
-   `6018` - SECONDARY_AUDIO_DEVICE_CONFLICT - Occurs in `stream.startSecondaryAudio` due to the microphone ID being used for session audio.
-   `6019` - SECONDARY_AUDIO_SCREEN_SHARING_CONFLICT - Occurs in `stream.startSecondaryAudio` due to screen sharing being in progress.
-   `6020` - SECONDARY_AUDIO_NOT_STARTED - Occurs in `stream.stopSecondaryAudio`, due to the secondary audio not being started.
-   `6021` - PSTN_MISMATCH_COUNTRY_CODE - Occurs in `stream.inviteByPhone` due to the country code does not match any code in the list.
-   `6022` - PSTN_NOT_CONNECTED - Occurs in `stream.hangup` due to the PSTN being not being connected.
-   `6023` - CRC_NOT_CONNECTED - Occurs in `stream.cancelCallCRCDevice` due to the CRC device not being connected.
-   `6026` - AUDIO_PROCESSOR_UNSUPPORTED - Occurs in `stream.createProcessor` due to the audio processor not being supported on the platform.
-   `6027` - AUDIO_PROCESSOR_DUPLICATE_CREATE - Occurs in `stream.createProcessor` due to the creation of a duplicate audio processor.
-   `6028` - AUDIO_PROCESSOR_MISMATCH_PROCESSOR - Occurs in `stream.addProcessor` and `stream.removeProcessor` because of the inability to find the target processor.
-   `6029` - AUDIO_PROCESSOR_LIMIT_EXCEED Occurs in the `stream.addProcessor` method due to adding the same processor again.

### Video exception

-   `6100` - VIDEO_CAMERA_PERMISSION_DENIED - Occurs in `stream.startVideo` due to the user being denied camera permission.
-   `6101` - VIDEO_NO_CAMERA - Occurs in `stream.startVideo` due to no camera.
-   `6102` - VIDEO_MISMATCH_CAMERA - Occurs in `stream.startVideo` due to the camera not matching the device list.
-   `6103` - VIDEO_CAMERA_TAKEN - Occurs in `stream.startVideo` due to the camera being taken by another program.
-   `6104` - VIDEO_VB_INITIALIZE_ERROR - Occurs in `stream.startVideo` due to a virtual background initialization error.
-   `6105` - VIDEO_CAPTURE_LOADING - Occurs in `stream.startVideo` or `stream.stopVideo` due to the video capture loading.
-   `6107` - VIDEO_VB_UNSUPPORTED - Occurs in `stream.startVideo` due to the virtual background not being supported on the platform.
-   `6108` - VIDEO_VB_PROCESSOR_CONFLICT - Occurs in `stream.startVideo` due to the virtual processor in progress.
-   `6109` - VIDEO_MISMATCH_STATE - Occurs in `stream.startVideo` or `stream.stopVideo` due to the video state not being matched.
-   `6110` - VIDEO_ESTABLISH_STREAM_ERROR - Occurs in `stream.startVideo` due to a video stream establish error.
-   `6111` - VIDEO_RENDER_LIMIT_EXCEED - Occurs in `stream.renderVideo` or `stream.attachVideo` due to number of rendered videos exceeding the limit.
-   `6112` - VIDEO_METHOD_DEPRECATED - Occurs in `stream.renderVideo` due to the method not being applicable for WebRTC video.
-   `6113` - VIDEO_VB_IMAGE_INACCESSIBLE - Occurs in `stream.previewVirtualBackground` due to the virtual background image being inaccessible.
-   `6114` - VIDEO_VB_MISMATCH_STATE - Occurs in `stream.previewVirtualBackground` due to the virtual background state not being matched.
-   `6115` - VIDEO_MASK_IMAGE_INACCESSIBLE - Occurs in `stream.previewMask` due to the mask image being inaccessible.
-   `6116` - VIDEO_MASK_PROCESSOR_CONFLICT - Occurs in `stream.previewMask` due to the virtual background or processor being in progress.
-   `6117` - VIDEO_SPOTLIGHT_MISMATCH_STATE - Occurs in `stream.spotlightVideo` due to video not started or spotlighted.
-   `6118` - VIDEO_SPOTLIGHT_MULTI_CONFLICT - Occurs in `stream.removeSpotlightedVideo` due to multiple spotlighted videos not being able to be removed.
-   `6119` - VIDEO_PROCESSOR_UNSUPPORTED - Occurs in `stream.createProcessor` due to the video processor being unsupported on the platform.
-   `6120` - VIDEO_PROCESSOR_LIMIT_EXCEED - Occurs in `stream.addProcessor` due to adding more than one video processor at the same time.
-   `6121` - VIDEO_PROCESSOR_DUPLICATE_CREATE - Occurs in `stream.createProcessor` due to creating duplicate video processors.
-   `6122` - VIDEO_PROCESSOR_MISMATCH_PROCESSOR - Occurs in `stream.addProcessor ` because of the inability to find the target processor.
-   `6150` - FAR_END_CAMERA_CONTROL_DEVICE_UNSUPPORTED - Occurs in `stream.controlCamera`, `stream.controlFarEndCamera`, or `stream.requestFarEndCameraControl` due to camera control being unsupported on the platform.
-   `6151` - FAR_END_CAMERA_CONTROL_MISMATCH_STATE - Occurs in `stream.requestFarEndCameraControl`, `stream.approveFarEndCameraControl`, `stream.declineFarEndCameraControl`, `stream.giveUpFarEndCameraControl`, or `stream.controlCamera` due to camera control mismatch.
-   `6152` - FAR_END_CAMERA_PERMISSION_REQUIRED - Occurs in `stream.controlFarEndCamera` because the camera control permission is not granted.
-   `6153` - FAR_END_CAMERA_INVALID_RANGE - Occurs in `stream.controlFarEndCamera` because the range is invalid.
-   `6154` - FAR_END_CAMERA_RATE_LIMIT - Occurs in `stream.controlFarEndCamera` because the method call rate has exceeded the limit.

### Screen share exception

-   `6200` - SCREEN_SHARE_USER_DENIED - Occurs in `stream.startShareScreen` because the user denied the screen share.
-   `6201` - SCREEN_SHARE_SYSTEM_DENIED - Occurs in `stream.startShareScreen` because the permission in the system setting is not allowed.
-   `6202` - SCREEN_SHARE_UNSUPPORTED - Occurs in `stream.startShareScreen` because the platform does not support the `getDisplayMedia` method.
-   `6203` - SCREEN_SHARE_SUBSESSION_CONFLICT - Occurs in `stream.startShareScreen` because the user attempted to share their screen to subsessions while already in a subsession.
-   `6204` - SCREEN_SHARE_HOST_PRIVILEGE_REQUIRED - Occurs in `stream.startShareScreen` because only the host has the share privilege.
-   `6205` - SCREEN_SHARE_MISMATCH_STATE- Occurs in `stream.startShareScreen`, `stream.shareToBreakoutRoom`, `stream.enableOptimizeForSharedVideo`, or `stream.muteOthersScreenShare` because the screen share state does not match.
-   `6206` - SCREEN_SHARE_OPTIMIZE_VIDEO_UNSUPPORTED - Occurs in `stream.startShareScreen` or `stream.enableOptimizeForSharedVideo` because the platform does not support the video share.
-   `6207` - SCREEN_SHARE_OPTIMIZE_VIDEO_CONFLICT - Occurs in `stream.enableOptimizeForSharedVideo` because the host has enabled multiple screen sharing.
-   `6208` - SCREEN_SHARE_INVALID_PRIVILEGE - Occurs in `stream.setScreenSharePrivilege` because the privilege is invalid.
-   `6220` - SCREEN_SHARE_WHITEBOARD_CONFLICT Occurs in `stream.startShareScreen` method because whiteboard sharing is currently in progress.

### Share annotation exception

-   `6210` - SCREEN_SHARE_ANNOTATION_ACCOUNT_DISABLE - Occurs in `stream.startAnnotation` because the account doesn't support annotation.
-   `6211` - SCREEN_SHARE_ANNOTATION_NO_ACTIVE_SHARE - Occurs in `stream.startAnnotation` because there is no active sharing session.
-   `6212` - SCREEN_SHARE_ANNOTATION_PRESENTER_DISABLED - Occurs in `stream.startAnnotation` because the sharing user disabled the annotation by `stream.changeAnnotationPrivilege`.
-   `6213` - SCREEN_SHARE_ANNOTATION_DUPLICATE_START - Occurs in `stream.startAnnotation` because annotation has already been started.
-   `6214` - SCREEN_SHARE_ANNOTATION_START_ERROR - Occurs in `stream.startAnnotation` because of failing to start annotation.
-   `6215` - SCREEN_SHARE_ANNOTATION_INCORRECT_PARAMETER - Occurs in `stream.getAnnotationController.clear` or `stream.getAnnotationController.setToolType` because the method parameter is invalid.
-   `6216` - SCREEN_SHARE_ANNOTATION_NOT_SUPPORT - Occurs in `stream.startAnnotation` because the current configuration doesn't support the annotation feature.
-   `6217` - SCREEN_SHARE_ANNOTATION_NOT_START - Occurs in `stream.stopAnnotation` because the annotation hasn't been started.
-   `6223` - SHARED_PROCESSOR_UNSUPPORTED - Occurs in `stream.createProcessor` method due to share processor being unsupported on the platform.
-   `6224` - SHARED_PROCESSOR_DUPLICATE_CREATE - Occurs in `stream.createProcessor` method due to a duplicate share processor creation.
-   `6225` - SHARED_PROCESSOR_MISMATCH_PROCESSOR - Occurs in the `stream.addProcessor` and `stream.removeProcessor` methods when unable to find the target processor.
-   `6226` - SHARED_PROCESSOR_LIMIT_EXCEED - Occurs in the `stream.addProcessor` method due to adding more than one (1) share processor at the same time.

## Chat exception

-   `7000` - CHAT_EMPTY_MESSAGE - Occurs in `chat.send` because the message is empty.
-   `7001` - CHAT_MISMATCH_USER - Occurs in `chat.send` because the user does not match.
-   `7002` - CHAT_INSUFFICIENT_PRIVILEGE - Occurs in `chat.send` because the privilege is not allowed.
-   `7003` - CHAT_INVALID_PRIVILEGE - Occurs in `chat.setPrivilege` because only the host or manager can set the privilege.
-   `7004` - CHAT_SUBSESSION_CONFLICT - Occurs in `chat.setPrivilege` or `chat.sendFilemethod` because the user is in a subsession.
-   `7005` - CHAT_EMPTY_FILE - Occurs in `chat.sendFile` because the file is empty.
-   `7006` - CHAT_INVALID_RETRY_TOKEN - Occurs in `chat.sendFile` because the retry token is invalid.
-   `7007` - CHAT_MISMATCH_FILE_TYPE - Occurs in `chat.sendFile` because the file is not in the allowed file type. See the account setting to update the allowed file types.
-   `7008` - CHAT_EXCEED_FILE_SIZE - Occurs in `chat.sendFile` because the file size exceed the maximum size. See the account setting to update the file size limit.
-   `7009` - CHAT_INVALID_MSG_ID_OR_URL - Occurs in `chat.downloadFile` because the message ID or file URL is invalid.
-   `7010` - CHAT_FILE_ACCOUNT_DISABLE - Occurs in `chat.sendFile` because the file transfer function is disabled in the account.
-   `7011` - CHAT_DELETE_ACCOUNT_DISABLE - Occurs in `chat.deleteMessage` because the delete chat message is disabled in the account.

## Command channel exception

-   `7100` - CMD_EMPTY_MESSAGE - Occurs in `cmd.send` because the message is empty.
-   `7101` - CMD_MISMATCH_USER - Occurs in `cmd.send` because the user does not match.
-   `7102` - CMD_INVALID_MESSAGE_TYPE - Occurs in `cmd.send` because the message type is not a string.
-   `7103` - CMD_EXCEED_MESSAGE_LENGTH - Occurs in `cmd.send` because the message length exceeded the maximum length of 512 bytes.
-   `7104` - CMD_CHANNEL_NOT_READY - Occurs in `cmd.send` because the channel is not ready.

## Recording exception

-   `7200` - RECORDING_ACCOUNT_DISABLE - Occurs in `recording.startCloudRecording` because cloud recording is not enabled in the account settings.
-   `7201` - RECORDING_STOP_FORBIDDEN - Occurs in `recording.startCloudRecording` because pausing or stopping cloud recording is forbidden in the account setting.
-   `7202` - RECORDING_MISMATCH_STATE - Occurs in `recording.startCloudRecording` because the recording state does not match.
-   `7203` - RECORDING_ISO_DISABLE - Occurs in `recording.acceptIndividualRecording` because individual recording is not enabled.

## Live transcription exception

-   `7300` - TRANSCRIPTION_ACCOUNT_DISABLE - Occurs in `liveTranscription.startLiveTranscription` because the feature is disabled in the account settings.
-   `7301` - TRANSCRIPTION_FEATURE_DISABLE - Occurs in `liveTranscription.startLiveTranscription` because the feature is disabled by the session host.
-   `7302` - TRANSCRIPTION_LANGUAGE_UNSUPPORTED - Occurs in `liveTranscription.setSpeakingLanguage` because the language is not supported.
-   `7303` - TRANSCRIPTION_MISMATCH_STATE - Occurs in `liveTranscription.setSpeakingLanguage` or `liveTranscription.setTranslationLanguage` because the transcription has not been supported.
-   `7304` - TRANSLATION_ACCOUNT_DISABLE - Occurs in `liveTranscription.setTranslationLanguage` because the feature is disabled in the account settings.
-   `7305` - TRANSLATION_LANGUAGE_UNSUPPORTED - Occurs in `liveTranscription.setTranslationLanguage` because the translation language is not supported.

## Live stream exception

-   `7400` - LIVE_STREAM_ACCOUNT_DISABLE - Occurs in `liveStream.startLiveStream` because the live stream is disabled in the account settings.
-   `7401` - LIVE_STREAM_INVALID_PARAMETER - Occurs in `liveStream.startLiveStream` because the parameter is invalid.
-   `7402` - LIVE_STREAM_MISMATCH_STATE - Occurs in `liveStream.startLiveStream` because the live stream is in progress.

## Subsession exception

-   `7500` - SUBSESSION_ACCOUNT_DISABLE - Occurs in `subsession.createSubsessions` because the subsession function is disabled in the account settings.
-   `7501` - SUBSESSION_EXCEED_CAPACITY - Occurs in `subsession.createSubsessions` because the number of created subsessions has exceeded the maximum limit of 100.
-   `7502` - SUBSESSION_MISMATCH_ID - Occurs in `subsession.openSubsessions`, `subsession.joinSubsession`, `subsession.assignUserToSubsession`, or `subsession.moveUserToSubsession` because the subsession ID is incorrect.
-   `7503` - SUBSESSION_MISMATCH_USER - Occurs in `subsession.openSubsessions`, `subsession.moveBackToMainSession`, `subsession.assignUserToSubsession`, or `subsession.moveUserToSubsession` because the user ID is incorrect.
-   `7504` - SUBSESSION_MISMATCH_STATE - Occurs in `subsession.openSubsessions` or other methods because the subsession status is not correct.
-   `7505` - SUBSESSION_EXIT_RESTRICTED - Occurs in `subsession.leaveSubsession` because the host restricts users from leaving the subsession.
-   `7506` - SUBSESSION_BROADCAST_DISABLE - Occurs in `subsession.broadcast` because the broadcast message function is disabled in the account.
-   `7507` - SUBSESSION_BROADCAST_INVALID_CONTENT - Occurs in `subsession.broadcast` because the message is empty or an invalid string.
-   `7508` - SUBSESSION_BROADCAST_VOICE_DISABLE - Occurs in `subsession.startBroadcastVoice` because broadcast voice is disabled in the account.
-   `7509` - SUBSESSION_BROADCAST_VOICE_RESTRICTED - Occurs in `subsession.startBroadcastVoice` because the host cannot broadcast voice within a subsession.
-   `7510` - SUBSESSION_BROADCAST_VOICE_MISMATCH_STATE - Occurs in `subsession.startBroadcastVoice` or `subsession.stopBroadcastVoice` because the host cannot broadcast voice without an active audio connection during the session.

## Broadcast streaming exception

-   `7900` - BROADCAST_STREAMING_ACCOUNT_DISABLE - Occurs in `broadcastStreamingClient.startBroadcast` because the feature is disabled in the account setting.
-   `7901` - BROADCAST_STREAMING_MISMATCH_STATE - Occurs in `broadcastStreamingClient.startBroadcast` because the status is incorrect.
-   `7902` - BROADCAST_STREAMING_INVALID_CHANNEL_ID - Occurs in `broadcastStreamingClient.stopBroadcast` because the channel ID in current case is required.

## Whiteboard exception

-   `8000` - WHITEBOARD_ACCOUNT_DISABLE - Occurs in `whiteboard.startWhiteboardScreen` method because the whiteboard feature is not enabled in the account setting.
-   `8001` - WHITEBOARD_UNSUPPORTED - Occurs in `whiteboard.startWhiteboardScreen` method because the whiteboard feature is not supported on the current platform (mobile browsers).
-   `8002` - WHITEBOARD_SCREEN_SHARE_CONFLICT - Occurs in `whiteboard.startWhiteboardScreen` method because screen sharing is currently in progress.
-   `8003` - WHITEBOARD_MISMATCH_STATE - Occurs in `whiteboard.startWhiteboardScreen` or `whiteboard.exportWhiteboard` methods because the whiteboard state does not match the required state.
-   `8004` - WHITEBOARD_HOST_PRIVILEGE_REQUIRED - Occurs in `whiteboard.startWhiteboardScreen` method because only the host or manager has permission to perform this action.
-   `8005` - WHITEBOARD_INVALID_PARAMETER - Occurs in `whiteboard.startWhiteboardScreen` or `whiteboard.stopWhiteboardScreen` or `whiteboard.startWhiteboardView` methods because the parameter is invalid (e.g., invalid document ID, template ID, or presenter ID).
-   `8006` - WHITEBOARD_EXPORT_DISABLE - Occurs in `whiteboard.exportWhiteboard` method because the whiteboard export feature has been disabled for the current session.

## Real-time media streams exception

-   `8100` - REAL_TIME_MEDIA_STREAMS_NOT_ENABLED - Occurs in `realTimeMediaStreamsClient.startRealTimeMediaStreams` method because the real-time media streams feature is not enabled in the account setting.
-   `8101` - REAL_TIME_MEDIA_STREAMS_INVALID_STATE - Occurs in `realTimeMediaStreamsClient.startRealTimeMediaStreams` or `realTimeMediaStreamsClient.stopRealTimeMediaStreams` methods because the real-time media streams state does not match the required state.
-   `8102` - REAL_TIME_MEDIA_STREAMS_NOT_SUPPORTED_IN_BREAKOUT_ROOM - Occurs in `realTimeMediaStreamsClient.startRealTimeMediaStreams` method because real-time media streams is not supported in breakout rooms.
