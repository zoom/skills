# Android Meeting SDK Environment Variables

| Variable | Required | Purpose | Where to find |
| --- | --- | --- | --- |
| `ZOOM_SDK_KEY` | Yes | Local alias for the current Meeting SDK Client ID (`appKey`) | Zoom Marketplace -> Meeting SDK app -> App Credentials |
| `ZOOM_SDK_SECRET` | Yes | Local alias for the current Meeting SDK Client Secret | Zoom Marketplace -> Meeting SDK app -> App Credentials |
| `ZOOM_MEETING_NUMBER` | Join/start | Meeting identifier | Zoom invite / web portal / Meetings API |
| `ZOOM_MEETING_PASSWORD` | Conditional | Meeting passcode | Zoom invite details / Meetings API |
| `ZOOM_ROLE` | Yes | Signature role (`0` attendee, `1` host) | App business logic |
| `ZOOM_ZAK` | Host start | Host authorization token | Zoom REST API token flow |

## Notes

- Generate signatures server-side only.
- Keep mobile client as consumer of short-lived tokens, not secret holder.
