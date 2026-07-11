# Zoom Meeting SDK Environment Variables

## Standard `.env` keys

| Variable | Required | Used for | Where to find |
| --- | --- | --- | --- |
| `ZOOM_CLIENT_ID` | Yes | Meeting SDK JWT `appKey` identity | Zoom Marketplace -> Meeting SDK app -> App Credentials |
| `ZOOM_CLIENT_SECRET` | Yes | Meeting SDK JWT signing secret | Zoom Marketplace -> Meeting SDK app -> App Credentials |
| `ZOOM_MEETING_NUMBER` | Per flow | Meeting to join/start | Zoom meeting invite, Zoom web portal, or Meetings API |
| `ZOOM_MEETING_PASSWORD` | Conditional | Meeting passcode | Meeting invite details / Meetings API |
| `ZOOM_ROLE` | Conditional | Signature role (`0` attendee, `1` host) | Set by your app logic |
| `ZOOM_ZAK` | Host flows only | Host authorization token for start flows | Generate via Zoom REST API user token endpoint |

## Runtime-only values

- `MEETING_SDK_JWT` (generated signature)

Generate server-side and keep short-lived.

## Notes

- Never expose `ZOOM_CLIENT_SECRET` in frontend/mobile clients.
- Existing samples can use variable names such as `ZOOM_SDK_KEY` and `ZOOM_SDK_SECRET`; treat
  those only as local aliases whose values now come from Client ID and Client Secret.

## Platform-Specific References

- Android: [../android/references/environment-variables.md](../android/references/environment-variables.md)
- iOS: [../ios/references/environment-variables.md](../ios/references/environment-variables.md)
- macOS: [../macos/references/environment-variables.md](../macos/references/environment-variables.md)
- Unreal: [../unreal/references/environment-variables.md](../unreal/references/environment-variables.md)
