# Marketplace App Templates

Use the JSON templates in `../assets/marketplace-apps/` to create a minimal app configuration
for a known scenario. Most files are POST-ready create requests. Templates marked as a base
require feature setup in Marketplace after creation because the public create or manifest schema
does not encode the complete feature.

## App-Type Compatibility

| Scenario | General user-managed | General admin-managed | S2S OAuth | Other required type |
|----------|----------------------|-----------------------|-----------|---------------------|
| User-scoped REST API | Yes | No | No | - |
| Account/admin REST API | No | Yes | Yes, same account only | - |
| OAuth authorization code or PKCE | Yes | Yes | No | - |
| Zoom Apps SDK | Yes | No | No | - |
| Plugin SDK | Yes | No | No | - |
| Meeting/Webinar RTMS, excluding ZCC | Yes, required | No | No | - |
| Contact Center Voice RTMS | Yes | Yes for admin scopes | Yes | - |
| Team Chat user API | Yes | No | No | - |
| Team Chat admin API | No | Yes | Yes | - |
| Team Chat chatbot | Conditional; supported in the UI but manifest validation has drift | Yes, reliable default | No | - |
| Webhook events | Yes for user events | Yes for account events | Yes for account events | - |
| WebSocket events | No; WebSockets are account-level | Yes | Yes | - |
| Default Zoom MCP server | Yes, recommended | Conditional by tool scope | Do not assume tool parity | - |
| Team Chat MCP server | Yes, required for current user tools | No | No | - |
| Whiteboard MCP server | Yes, verified path | No | Discovery only until tool execution is proven | - |
| Meeting SDK | No | No | No | Dedicated `meeting_sdk` app |

Treat scope suffixes as a hard boundary. User-managed General Apps use user scopes. Admin-managed
General Apps and S2S apps use account/admin scopes. Do not combine both scope classes in one app.

Meeting/Webinar RTMS is specifically user-managed. Contact Center Voice RTMS is different and
can use General or S2S depending on whether the workflow acts for a user or the account. Plugin
SDK requires a user OAuth token matching the user signed in to Zoom Workplace.

## Template Selector

| Scenario | Template | App model | Primary skill |
|----------|----------|-----------|---------------|
| S2S Meeting/account API | [create-s2s-oauth.json](../assets/marketplace-apps/create-s2s-oauth.json) | Server-to-Server OAuth | [OAuth](../../oauth/SKILL.md) |
| S2S Contact Center API | [create-s2s-contact-center-api.json](../assets/marketplace-apps/create-s2s-contact-center-api.json) | Server-to-Server OAuth | [Contact Center](../../contact-center/SKILL.md) |
| S2S Phone API | [create-s2s-phone-api.json](../assets/marketplace-apps/create-s2s-phone-api.json) | Server-to-Server OAuth | [Phone](../../phone/SKILL.md) |
| S2S Team Chat admin API | [create-s2s-team-chat-api.json](../assets/marketplace-apps/create-s2s-team-chat-api.json) | Server-to-Server OAuth | [Team Chat](../../team-chat/SKILL.md) |
| S2S webhook event base | [create-s2s-webhooks.json](../assets/marketplace-apps/create-s2s-webhooks.json) | Server-to-Server OAuth, then Access UI | [Webhooks](../../webhooks/SKILL.md) |
| S2S WebSocket event base | [create-s2s-websocket.json](../assets/marketplace-apps/create-s2s-websocket.json) | Server-to-Server OAuth, then Access UI | [WebSockets](../../websockets/SKILL.md) |
| S2S Contact Center Voice RTMS | [create-s2s-zcc-voice-rtms.json](../assets/marketplace-apps/create-s2s-zcc-voice-rtms.json) | Server-to-Server OAuth, then RTMS events | [RTMS](../../rtms/SKILL.md) |
| Embed Zoom meetings | [create-meeting-sdk.json](../assets/marketplace-apps/create-meeting-sdk.json) | Meeting SDK | [Meeting SDK](../../meeting-sdk/SKILL.md) |
| User-authorized REST API | [general-user-api.json](../assets/marketplace-apps/general-user-api.json) | General App, user-managed | [REST API](../SKILL.md) |
| Account-wide REST API | [general-admin-api.json](../assets/marketplace-apps/general-admin-api.json) | General App, admin-managed | [REST API](../SKILL.md) |
| In-client Zoom App | [zoom-app.json](../assets/marketplace-apps/zoom-app.json) | General App with Zoom Apps surface | [Zoom Apps SDK](../../zoom-apps-sdk/SKILL.md) |
| Native Plugin SDK companion | [plugin-sdk.json](../assets/marketplace-apps/plugin-sdk.json) | General App, user-managed with PKCE | [Plugin SDK](../../plugin-sdk/SKILL.md) |
| User-authorized Team Chat API | [team-chat-api.json](../assets/marketplace-apps/team-chat-api.json) | General App, user-managed | [Team Chat](../../team-chat/SKILL.md) |
| Admin-authorized Team Chat API | [team-chat-admin-api.json](../assets/marketplace-apps/team-chat-admin-api.json) | General App, admin-managed | [Team Chat](../../team-chat/SKILL.md) |
| Team Chat chatbot | [team-chat-chatbot.json](../assets/marketplace-apps/team-chat-chatbot.json) | General App with Chat subscription | [Team Chat](../../team-chat/SKILL.md) |
| Contact Center admin API | [contact-center.json](../assets/marketplace-apps/contact-center.json) | General App, admin-managed | [Contact Center](../../contact-center/SKILL.md) |
| Zoom Phone API | [phone-api.json](../assets/marketplace-apps/phone-api.json) | General App, admin-managed | [Phone](../../phone/SKILL.md) |
| User meeting webhooks | [general-user-webhooks.json](../assets/marketplace-apps/general-user-webhooks.json) | General App, user-managed | [Webhooks](../../webhooks/SKILL.md) |
| Meeting webhooks | [meeting-webhooks.json](../assets/marketplace-apps/meeting-webhooks.json) | General App with event subscription | [Webhooks](../../webhooks/SKILL.md) |
| General App WebSocket event base | [general-admin-websocket.json](../assets/marketplace-apps/general-admin-websocket.json) | General App admin, then Access UI | [WebSockets](../../websockets/SKILL.md) |
| Meeting/Webinar RTMS, excluding ZCC | [general-user-meeting-webinar-rtms.json](../assets/marketplace-apps/general-user-meeting-webinar-rtms.json) | General App user, then RTMS events | [RTMS](../../rtms/SKILL.md) |
| Contact Center Voice RTMS | [zcc-voice-rtms.json](../assets/marketplace-apps/zcc-voice-rtms.json) | General App with ZCC RTMS events | [RTMS](../../rtms/SKILL.md) |
| Default Zoom MCP server | [zoom-mcp-default.json](../assets/marketplace-apps/zoom-mcp-default.json) | General App, user-managed with PKCE | [Zoom MCP](../../zoom-mcp/SKILL.md) |
| Team Chat MCP server | [zoom-mcp-team-chat.json](../assets/marketplace-apps/zoom-mcp-team-chat.json) | General App, user-managed with PKCE | [Team Chat MCP](../../zoom-mcp/team-chat/SKILL.md) |
| Whiteboard MCP server | [zoom-mcp-whiteboard.json](../assets/marketplace-apps/zoom-mcp-whiteboard.json) | General App, user-managed with PKCE | [Whiteboard MCP](../../zoom-mcp/whiteboard/SKILL.md) |

## Skill Coverage Audit

Every top-level product skill now has either a concrete template route or an explicit reason
that Marketplace app generation does not apply. Platform child skills inherit their parent
product's app model unless the child skill states otherwise.

| Skill | Marketplace setup coverage |
|-------|----------------------------|
| `general` | Routes to the selector and generic General user, General admin, and S2S templates. |
| `oauth` | Covered by generic General user, General admin, and S2S templates; select by grant and scope ownership. |
| `rest-api` | Covered by generic API templates plus Contact Center, Phone, Team Chat, webhook, and WebSocket variants. |
| `meeting-sdk` | Uses the dedicated [Meeting SDK create request](../assets/marketplace-apps/create-meeting-sdk.json). All platform children inherit it. |
| `video-sdk` | Uses Build Platform SDK credentials, not a supported Marketplace app-generation payload. All platform children inherit it. |
| `ui-toolkit` | Uses the same Build Platform Video SDK app and session JWT as `video-sdk`. |
| `cobrowse-sdk` | Uses the Build Platform app's Cobrowse credentials; do not create a General App manifest for the SDK session. |
| `probe-sdk` | Requires no Zoom Marketplace credentials for core diagnostics. |
| `zoom-apps-sdk` | Covered by [zoom-app.json](../assets/marketplace-apps/zoom-app.json). |
| `plugin-sdk` | Covered by [plugin-sdk.json](../assets/marketplace-apps/plugin-sdk.json). |
| `rtms` | Covered for Meeting/Webinar General App, ZCC Voice General/S2S, and Build Platform Video SDK RTMS. |
| `team-chat` | Covered for user API, admin API, S2S API, and chatbot scenarios. |
| `webhooks` | Covered for General user, General admin, and S2S starters; Build Platform events remain configured on the Build app. |
| `websockets` | Covered by General admin and S2S base templates. |
| `phone` | Covered by General admin and S2S Phone API templates; user-level Phone calls should start from the generic user template with exact endpoint scopes. |
| `contact-center` | Covered by General admin and S2S API templates; browser/mobile channel embeds also require Contact Center entry IDs or API keys from product admin. |
| `virtual-agent` | KB automation starts from generic General admin or S2S; web/mobile embeds use Virtual Agent campaign keys and entry IDs instead. |
| `rivet-sdk` | Composite framework; choose the template for the underlying REST, OAuth, webhook, Team Chat, or Video SDK module. |
| `scribe` | Uses Build Platform API key/secret JWT authentication, not a General/S2S Marketplace template. |
| `summarizer` | Uses Build Platform API key/secret JWT authentication, not a General/S2S Marketplace template. |
| `translator` | Uses Build Platform API key/secret JWT authentication, not a General/S2S Marketplace template. |
| `zoom-mcp` | Covered by separate default, Team Chat, and Whiteboard user-managed General App templates with PKCE. |

## Create Workflow

1. Select the narrowest matching template.
2. Replace all `example.com` URLs, app names, descriptions, domains, and contact fields.
3. Check every OAuth scope against the exact target API operation. Remove unused scopes.
4. For General Apps, extract and validate the inner manifest:

```bash
jq '{manifest: .manifest}' TEMPLATE.json > /tmp/manifest-validation.json

curl -X POST "https://api.zoom.us/v2/marketplace/apps/manifest/validate" \
  -H "Authorization: Bearer $ZOOM_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  --data @/tmp/manifest-validation.json
```

5. Check both HTTP status and `ok`; validation can return HTTP `200` with `ok: false`.
6. Create General Apps through the regular endpoint:

```bash
curl -X POST "https://api.zoom.us/v2/marketplace/apps" \
  -H "Authorization: Bearer $ZOOM_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  --data @TEMPLATE.json
```

Create S2S and Meeting SDK apps through the account-scoped endpoint with
`marketplace:write:app:master`:

```bash
curl -X POST "https://api.zoom.us/v2/accounts/$ZOOM_ACCOUNT_ID/marketplace/apps" \
  -H "Authorization: Bearer $ZOOM_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  --data @TEMPLATE.json
```

7. Store returned client secrets in a secret manager. Never print, commit, or retain them in
   test artifacts.
8. Fetch the created app or exported General App manifest and compare it with the intended
   configuration because Marketplace can normalize or omit fields.

## Required Post-Create Setup

- **S2S and General WebSocket bases:** Open Features > Access, enable Event Subscription,
  choose the WebSocket method, add account-level events, save, and copy the generated WebSocket
  endpoint URL. The published manifest schema does not currently expose a reliable one-shot
  WebSocket delivery field.
- **S2S webhook base:** Open Features > Access, enable Event Subscription, add the exact
  account-level events and webhook receiver URL, and retain the generated secret token for
  request validation. The native S2S create request carries scopes but not the complete event
  subscription definition.
- **General user-managed Meeting/Webinar RTMS:** Add the `meeting.rtms_started`, `meeting.rtms_stopped`,
  `meeting.rtms_interrupted`, `webinar.rtms_started`, `webinar.rtms_stopped`, and
  `webinar.rtms_interrupted` events in Marketplace. Current manifest create validation has
  rejected these event names even though saved apps can contain them.
- This Meeting/Webinar template is not for Zoom Contact Center. Use the separate General or S2S
  Contact Center Voice RTMS templates for `contact_center.voice_rtms_*` events.
- **S2S Contact Center Voice RTMS:** Add Contact Center Voice RTMS started, stopped, and
  interrupted events after creation. Confirm the selected media and control scopes before
  activating the app.

## Guardrails

- `s2s_oauth` and `meeting_sdk` are native create request types; they do not use a General App
  manifest.
- Runtime testing on 2026-07-10 required S2S and Meeting SDK creation through
  `/v2/accounts/{accountId}/marketplace/apps`; the regular endpoint redirected callers to that
  route despite broader wording in the public API description.
- General App `USER_OPERATION` accepts user scopes; `ADMIN_MANAGEMENT` accepts admin scopes.
- App-owned scopes such as `marketplace:write:event_subscription` do not belong in
  `oauth_information.scopes`; obtain them through `client_credentials` after app creation.
- Webhook-only behavior is configured as a General App event subscription. Do not use
  undocumented `webhook` or `webhook_only` app types.
- Team Chat chatbot command names must be unique. Change the template command before cloning.
- `domain_allow_list` entries are objects with `domain` and `explanation`.
- Use `WEB_SOCKET` for WebSocket event delivery where that field is supported.
- Do not place `marketplace:write:websocket_connection` in a General OAuth manifest. It is an
  app-owned scope obtained with the token used for WebSocket operations.
- Meeting/webinar RTMS event names can drift between saved manifests and validation. Use the
  Marketplace UI or event-subscription API when current validation rejects a documented event.
- Product licensing, account feature gates, app authorization, and Marketplace review can still
  block a schema-valid template.
- Plugin SDK is new and can be entitlement-gated. The `plugin_sdk.enable` shape was accepted by
  manifest validation, but the account must expose the Plugin SDK feature before the app can use it.
- Zoom MCP uses a manually registered General App and OAuth authorization-code flow. Keep PKCE
  enabled, replace the sample redirect URI with the MCP client's exact callback URL, and remove
  any tool scopes the connector will not use.

See [Marketplace App Management](marketplace-apps.md) for response shapes, validation quirks,
credential handling, and event-subscription sequencing.
