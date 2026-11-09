# PKCE Public-Client Authentication (No JWT, No Secret)

Verified end-to-end pattern for authenticating the Meeting SDK for Windows with
OAuth Authorization Code + PKCE using a public client id. No JWT, no SDK
key/secret, no backend. Live-verified against Meeting SDK 7.0.2 (April 2026+
PKCE support) with a real meeting start.

Based on the official guide:
https://developers.zoom.us/docs/meeting-sdk/windows/start-join-mtg-webinar/pkce/

---

## When to use this vs JWT

| | JWT path | PKCE public-client path |
|---|---|---|
| Credentials | SDK Client ID + Secret | OAuth client id only (public) |
| SDK auth call | `SDKAuth(jwt_token)` | `SDKAuth(publicAppKey)` |
| User identity | Email login / SSO | OAuth consent in browser |
| Server required | To sign the JWT | None |
| Best for | Embedded bots, kiosk | User-facing desktop apps |

---

## Critical: which client id

A Meeting SDK General app has FOUR client ids once PKCE is enabled
(`credentials.privateClient`, `publicClient`, `pkcePrivateClient`,
`pkcePublicClient` in the app-detail API):

- **Header-less PKCE token exchanges MUST use the PKCE-slot client id**
  (`pkcePrivateClient.clientId` for development). The classic dev client id is
  a confidential client and rejects header-less exchanges with
  `{"error":"invalid_client"}`.
- Enable it in the Marketplace UI: app -> Credentials/Feature -> enable
  **public client OAuth / PKCE** for development (or PATCH
  `enableDevPkceClient: true` via the app API). Enabling it issues the PKCE
  client ids; no separate save step.
- The production ids do not work until the app is published and approved.
- The same PKCE client id is also the SDK `publicAppKey` (see below). One id,
  two consumers (OAuth layer + SDK auth layer).

---

## Flow

```
1. Generate code_verifier (43+ chars) + S256 code_challenge
2. Browser: GET zoom.us/oauth/authorize?response_type=code
     &client_id=<PKCE id>&code_challenge=<S256>&code_challenge_method=S256
     &redirect_uri=<registered URL, byte-exact incl. query string>
   User clicks Allow -> redirect with ?code=...
3. POST zoom.us/oauth/token  (NO Authorization header)
     grant_type=authorization_code & code & client_id & redirect_uri
     & code_verifier
   -> { access_token, ... }
4. GET api.zoom.us/v2/users/me/token?type=zak
     Authorization: Bearer <access_token>
   -> { token: <ZAK> }   # ~2h lifetime
5. InitSDK -> CreateMeetingService -> SDKAuth(publicAppKey=<PKCE id>)
6. On AUTHRET_SUCCESS:
     - no passcode: Start(SDK_UT_WITHOUT_LOGIN, userZAK=<ZAK>, ...)
     - passcode-protected meeting: Join(JoinParam4WithoutLogin
       userZAK=<ZAK>, psw=<passcode>, ...)  # host join starts it
```

---

## Gotchas (all hit in live verification)

1. **redirect_uri must be byte-identical** - including any query string -
   between the authorize request and the token exchange. A mismatch returns
   HTTP 400 `invalid_client` (Zoom's generic wrapper; the dedicated error is
   4709 "Redirect URI mismatch"). Derive both from one config value.
2. **No `Authorization: Basic` on the token exchange** for the public-client
   flow. The `code_verifier` IS the proof; client_id travels in the form body.
   Sending Basic with an empty secret also fails.
3. **Authorization codes are single-use and short-lived (~2 min).** A retry
   after a failed exchange needs a fresh consent; the code is burned.
4. **`StartParam4WithoutLogin` has NO passcode member** (SDK 7.0.2: userZAK,
   userName, zoomuserType, meetingNumber, vanityID, customer_key, AV flags).
   For passcode-protected meetings, `Start()` is accepted but the meeting
   never opens (status callbacks stall). Use the `JoinParam4WithoutLogin`
   path instead - it has `psw` AND `userZAK`; host join via ZAK + psw starts
   the meeting.
5. **`AuthContext.publicAppKey`** (auth_service_interface.h) is documented as
   "Public app key used for SDK authentication. Alternative to JWT token" -
   set it to the PKCE client id. There is no separate SDK public key field in
   the app record; the four client-id slots are the entire credential surface.
6. **The authorize page returns HTTP 200 for any syntactically valid client
   id** - a 200 there proves nothing about token-endpoint validity. Debug
   `invalid_client` by checking WHICH client id and publication status first.
7. **The ZAK is a per-run credential (~2 h)** - refetch each session; the
   refresh token can mint new access tokens without a new consent.

---

## Minimal C++ sketch (SDK 7.0.2)

```cpp
// SDK auth - public client, no JWT
AuthContext authContext;
authContext.publicAppKey = pkceClientIdW.c_str();  // the PKCE client id
authService->SDKAuth(authContext);
// ... on AUTHRET_SUCCESS:

if (passcodeW.empty()) {
    StartParam startParam;
    startParam.userType = SDK_UT_WITHOUT_LOGIN;
    auto& s = startParam.param.withoutloginStart;
    s.userZAK = zakW.c_str();
    s.userName = userNameW.c_str();
    s.zoomuserType = ZoomUserType_EMAIL_LOGIN;
    s.meetingNumber = meetingNumber;
    meetingService->Start(startParam);
} else {
    JoinParam joinParam;
    joinParam.userType = SDK_UT_WITHOUT_LOGIN;
    auto& j = joinParam.param.withoutloginuserJoin;
    j.userZAK = zakW.c_str();
    j.userName = userNameW.c_str();
    j.meetingNumber = meetingNumber;
    j.psw = passcodeW.c_str();   // host join via ZAK starts the meeting
    meetingService->Join(joinParam);
}
```

A complete working reference implementation (PKCE pair generation via
`CryptBinaryToStringA`/`BCrypt`, manual + loopback callback modes, chunked
source writes, config-driven) exists as `PKCEAuthDemo` alongside the SDK
demos; its README documents the config contract.
