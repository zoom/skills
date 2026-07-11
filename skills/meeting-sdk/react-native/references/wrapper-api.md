# Wrapper API Reference (React Native package)

## Init config

- `jwtToken?: string`
- `domain?: string`
- `enableLog?: boolean`
- `logSize?: number` (Android)
- `bundleResPath?: string` (iOS)
- `appGroupId?: string` (iOS)
- `replaykitBundleIdentifier?: string` (iOS)

## Methods

- `initSDK(config): Promise<boolean>`
- `isInitialized(): Promise<boolean>`
- `joinMeeting(config): Promise<MobileRTCMeetError>` (`7.0.5+`)
- `startMeeting(config): Promise<MobileRTCMeetError>` (`7.0.5+`)
- `muteMyVideo(mute): Promise<MobileRTCSDKError>` (`7.0.5+`)
- `muteMyAudio(mute): Promise<MobileRTCAudioError | MobileRTCSDKError>` (`7.0.5+`)
- `connectMyAudio(on): Promise<boolean>` (`7.0.5+`)
- `updateMeetingSetting(config): void`
- `cleanup(): void`

## Join config highlights

- Required: `userName`, `meetingNumber`
- Optional: `password`, `zoomAccessToken`, `vanityID`, `webinarToken`, `joinToken`, `appPrivilegeToken`

## Start config highlights

- Required: `userName`, `zoomAccessToken`
- Optional: `meetingNumber`, `vanityID`, `inviteContactId`
