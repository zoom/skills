<!-- Source: https://developers.zoom.us/docs/video-sdk/web/sip.md -->
<!-- Fetched: 2026-06-15 -->

# Room (SIP)


Video SDK offers the ability to call out to conference room devices using the H.323/SIP [Conference Room Connector](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0060661). _H.323 and Session Initiation Protocol (SIP) are protocols for sending media online._

## Prerequisites

-   [Video SDK Account](https://zoom.us/pricing/developer).
-   [Conference Room Connector Plan](https://zoom.us/billing).
-   **The session must exist** - there must be at least one user in the session before you can use call in or call out.

## Dial out

Use `callCRCDevice` to call out to a Cloud Room Connector (CRC) device to join a session.

```javascript
stream.callCRCDevice("7357@test.plcm.vc", 2);
```

### Cancel room device call

Cancel the CRC call with `cancelCallCRCDevice` if you need to cancel the call out request before it is complete.

```javascript
stream.cancelCallCRCDevice("7357@test.plcm.vc", 2);
```

### Event listener for call status

Listen for the CRC device call state changes with the `crc-call-out-state-change` event. States include `busy`, `fail`, `ringing`, `success`, `timeout`, and `unreachable`.

```javascript
client.on("crc-call-out-state-change", (payload) => {
    console.log(payload.code);
});
```

### Webhooks

Use the following webhooks to get notified of the call out status.

-   [`session.user_room_system_callout_accepted`](/docs/api/video-sdk/events/#tag/session/POSTsession.user_phone_callout_accepted)
-   [`session.user_room_system_callout_failed`](/docs/api/video-sdk/events/#tag/session/POSTsession.user_room_system_callout_failed)
-   [`session.user_room_system_callout_missed`](/docs/api/video-sdk/events/#tag/session/POSTsession.user_phone_callout_missed)
-   [`session.user_room_system_callout_rejected`](/docs/api/video-sdk/events/#tag/session/POSTsession.user_phone_callout_rejected)
-   [`session.user_room_system_callout_ringing`](/docs/api/video-sdk/events/#tag/session/POSTsession.user_room_system_callout_ringing)

## Dial-in

To dial in to a Video SDK session, append the session number to `@zoomcrc.com`, for example `{SESSION_NUMBER}@zoomcrc.com`, and provide the passcode when prompted. You can obtain these values using an SDK function or API endpoints.

### Reserve a session to set up dial-in access

To enable dial-in, you must reserve a session ID in advance using the [Create a session API](/docs/api/video-sdk/#tag/sessions/post/videosdk/sessions). This reserves the session for up to 24 hours for your specified session name. When you start and join the session with that session name during the reservation period, dial-in users will be able to access it.

### Get dial-in information

Get dial-in details using either the SDK function or API endpoints.

#### Using an SDK function

To retrieve dial-in information with the `stream.getSessionSIPAddress()` function, the returned value will be in this format:
`{SESSION_NUMBER}.{PASSCODE}@global.zoomcrc.com`

For example:

```javascript
var sipAddress = stream.getSessionSIPAddress();
```

#### Using API Endpoints

Get dial-in information using either the [Create a session](/docs/api/video-sdk/#tag/sessions/post/videosdk/sessions) session or [Get session details](/docs/api/video-sdk/#tag/sessions/get/videosdk/sessions/{sessionId}) APIs.

**Sample request**

```javascript
var sipDialInAddress = responseBody.session_number + "@zoomcrc.com";
var sipDialInPasscode = responseBody.session_password;
```

**Sample response**

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
        "global_dial_in_countries": [
            //...
        ],
        "global_dial_in_numbers": [
            //...
        ]
    }
}
```

## Custom welcome message

You can customize the welcome message that H.323/SIP participants hear when they join a session. See [H.323/SIP Conferencing](/docs/build/account/#h323sip-conferencing) in the account settings documentation for details.

## More SIP features

For the full set of SIP features, see [Stream in the Video SDK Reference](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html).

## From the developer blog

For more complex SIP flows, see the following blog post.

-   [How to use Twilio IVR Dial Trees with the Zoom Video SDK](/blog/ivr-dial-trees-with-video-sdk) by Tommy Gaessler - 02-06-2024
