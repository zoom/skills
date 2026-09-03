<!-- Source: https://developers.zoom.us/docs/video-sdk/web/pstn.md -->
<!-- Fetched: 2026-06-15 -->

# Phone (PSTN)


Video SDK offers Public Switched Telephone Network (PSTN) features so that the SDK can dial out to people to join Video SDK sessions completely over the phone or dial-in to use their phone as the audio connection to the session while viewing the Video on Web. This is useful when:

-   You do not have a microphone or speaker on your computer.
-   You do not have an iOS or Android smartphone.
-   You cannot connect to a network for a video session with computer audio (VoIP).

See [Customize audio conferencing](/docs/video-sdk/account/#customize-audio-conferencing) for details.

## Prerequisites

-   [Video SDK Account](https://zoom.us/pricing/developer)
-   [Audio Conferencing Plan](https://zoom.us/billing)
-   **The session must exist**

## Dial out

Video SDK offers the ability for people to join a session by phone. While in a session, you can dial out to others so they can join by phone.

### Set up greeting

If you'd like to use a greeting, set up a **Telephone welcome message** in your account web portal under **Account Settings** and choose to require a [greeting](https://marketplacefront.zoom.us/sdk/custom/web/interfaces/ZoomVideo.DialOutOption.html#greeting) when you initiate the call. If you're using the Video SDK API, set `invite_options` `require_greeting` to `true` in [Use in-session events controls](/docs/api/video-sdk/#tag/sessions/PUT/videosdk/sessions/{sessionId}/status).

### Make a call

Use `stream.inviteByPhone()` to make a call.

-   Provide the name if the person will join completely by phone.
-   If the person is already connected to the session on the web, but wants to use their phone for the audio portion, the name will default to the name provided in the `client.join()` function.

```javascript
stream.inviteByPhone("+1", "2025550176", "Jane Dev");
```

For an extension, use a hyphen with the extension number, for example, for extension "123" use the following.

```javascript
stream.inviteByPhone("+1", "2025550176-123", "Jane Dev");
```

### Hang up a call

Use `stream.hangup()` to hang up a call.

```javascript
stream.hangup();
```

### Event listener

Use the `dialout-state-change` event to find out about the status of the call, for example, ringing, accepted, hangup, etc.

```javascript
client.on("dialout-state-change", (payload) => {
    console.log(payload);
});
```

### Webhooks

You can also use the following [webhooks](/docs/api/video-sdk/events/#tag/session) to get notified of the dial out status.

-   `session.user_phone_callout_accepted`
-   `session.user_room_system_callout_failed`
-   `session.user_phone_callout_missed`
-   `session.user_phone_callout_rejected`
-   `session.user_phone_callout_ringing`

## Dial-in

People can dial in to the session once it has been created. This is currently available only for Zoom-provided dial-in numbers. You cannot currently use your own phone exchange, known as Bring Your Own Carrier (BYOC). This operates similarly to Zoom Meetings [dial-in numbers](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0060564#h_01FGC34YME1ZT1TY2QKQB5NA2N).

### Reserve a session to set up dial-in access

To enable dial-in, you must reserve a session ID in advance using the [Create a session API](/docs/api/video-sdk/#tag/sessions/post/videosdk/sessions). This reserves the session for up to 24 hours for your specified session name. When you start and join the session with that session name during the reservation period, dial-in users will be able to access it.

### Get dial-in information

Get dial-in details using either an SDK function or API endpoints.

#### Using an SDK function

Get dial-in values with the `stream.getCurrentSessionCallinInfo()` function.

```javascript
var meetingId = stream.getCurrentSessionCallinInfo().meetingId;
var password = stream.getCurrentSessionCallinInfo().password;
var tollNumbers = stream.getCurrentSessionCallinInfo().tollNumbers;
```

#### Using API endpoints

Get dial-in information using either the [Create a session](/docs/api/video-sdk/#tag/sessions/post/videosdk/sessions) session or [Get session details](/docs/api/video-sdk/#tag/sessions/get/videosdk/sessions/{sessionId}) APIs.

**Sample API Response**

```json
{
    "session_id": "sfk/aOFJSJSYhGwk1hnxgw==",
    "session_number": 97763643886,
    "session_name": "My session",
    "session_password": "123456",
    "passcode": "123456",
    "created_at": "2022-03-25T07:29:29Z",
    "settings": {
        "auto_recording": "cloud",
        "global_dial_in_countries": ["US"],
        "global_dial_in_numbers": [
            {
                "country": "US",
                "country_name": "US",
                "number": "+1 1000200200",
                "type": "toll"
            }
        ]
    }
}
```

## More PSTN features

For the full set of PSTN features, see [Stream in the Video SDK Reference](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html).

## From the developer blog

For more complex PSTN flows, see the following blog post.

-   [How to use Twilio IVR Dial Trees with the Zoom Video SDK](/blog/ivr-dial-trees-with-video-sdk) by Tommy Gaessler - 02-06-2024
