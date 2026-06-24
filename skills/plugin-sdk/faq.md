# Plugin SDK FAQ

## How is Plugin SDK different from Meeting SDK for Windows or macOS?

Use **Plugin SDK** when a native desktop companion app controls the user's installed Zoom Workplace client over IPC.

Use **Meeting SDK for Windows or macOS** when the application embeds Zoom meeting functionality inside its own app process.

| Question | Plugin SDK | Meeting SDK for Windows/macOS |
|---|---|---|
| What does the app control? | The official installed Zoom Workplace client | An embedded Zoom meeting client inside your app |
| Who owns the meeting UI? | Zoom Workplace owns the UI | Your app owns or embeds the meeting UI |
| Is Zoom Workplace required? | Yes, compatible Zoom Workplace is required | No, the app ships Meeting SDK runtime |
| Does the app render media? | No, Zoom Workplace renders media | Yes, the SDK can render or expose media |
| Is this a bot path? | No, it controls a user's client | Windows can support headless bot patterns |
| Is this for custom UI? | Limited to supported Workplace UI actions | Yes, default or custom UI patterns |
| Is this for raw media? | No raw media transport responsibility | Use Meeting SDK raw data paths |
| Best share use case? | Share known app/window via official client | Share from embedded meeting implementation |
| Upgrade behavior? | Follows installed Workplace compatibility | You ship and upgrade SDK runtime |
| Primary auth shape? | User OAuth plus Plugin SDK scope | Meeting SDK auth/signature patterns |

## When should routing choose Plugin SDK?

- Choose Plugin SDK for native companion apps.
- Choose Plugin SDK for controlling Zoom Workplace.
- Choose Plugin SDK for official-client workflows.
- Choose Plugin SDK for Share-to-Zoom buttons.
- Choose Plugin SDK for Workplace IPC integrations.
- Choose Plugin SDK for standard Zoom UI.
- Choose Plugin SDK when users stay in Workplace.

## When should routing choose Meeting SDK?

- Choose Meeting SDK for embedded meetings.
- Choose Meeting SDK for custom meeting UI.
- Choose Meeting SDK for SDK-rendered meeting windows.
- Choose Meeting SDK for raw media access.
- Choose Meeting SDK for headless meeting bots.
- Choose Meeting SDK when shipping meeting runtime.
- Choose Meeting SDK when replacing Workplace UI.

## Common ambiguity

If the user asks for a native app that "starts Zoom" or "shares a specific local window in Zoom," prefer Plugin SDK if the target is the already installed Zoom Workplace client.

If the user asks to "embed Zoom meeting into my Windows/macOS app," "build custom meeting UI," "capture raw audio/video," or "run a bot," prefer Meeting SDK for the relevant platform.

If both signals appear, ask:

```text
Should the app control the installed Zoom Workplace client, or should it embed Zoom meeting functionality inside the app?
```
