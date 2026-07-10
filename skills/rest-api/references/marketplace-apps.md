# Zoom Marketplace API

Authoritative endpoint inventory for Marketplace. This file mirrors the official Zoom API Hub OpenAPI document for this product area.

## Canonical Source

- OpenAPI JSON: https://developers.zoom.us/api-hub/marketplace/methods/endpoints.json
- Base URL: `https://api.zoom.us/v2`
- Authentication details: [authentication.md](authentication.md)

## Notes

- Endpoint methods and paths below are generated from the official Zoom API Hub `paths` object.
- Scope names are defined per operation and frequently use granular scope names. Check the API Hub operation page for the exact scopes before implementation.
- Use this file for endpoint discovery and inventory. Use `../examples/` for orchestration patterns, not as the canonical source of path names.

## Auth Caveat: App-Owned Marketplace Scopes

Some Marketplace app-management scopes require an app-owned access token from the OAuth
`client_credentials` grant, not a user OAuth token and not a Server-to-Server
`account_credentials` token.

Use this flow when the Marketplace operation calls for scopes such as:

| Scope | Typical use |
|-------|-------------|
| `marketplace:write:event_subscription` | Create app event subscriptions |
| `marketplace:read:list_event_subscriptions` | List app event subscriptions |
| `marketplace:update:event_subscription` | Update or subscribe event subscriptions |
| `marketplace:delete:event_subscription` | Delete or unsubscribe event subscriptions |
| `marketplace:write:websocket_connection` | Create app-owned WebSocket connections |

Token request:

```bash
curl -X POST "https://zoom.us/oauth/token?grant_type=client_credentials" \
  -H "Authorization: Basic $(printf '%s:%s' "$ZOOM_CLIENT_ID" "$ZOOM_CLIENT_SECRET" | base64)" \
  -H "Content-Type: application/x-www-form-urlencoded"
```

Implementation rule:
- Use `client_credentials` for app-owned Marketplace scopes listed above.
- Use `account_credentials` only for Server-to-Server OAuth account automation scopes.
- Use `authorization_code` only when the API should act on behalf of an installing user.

## App Creation, Manifest, and Response Shapes

Use these notes when creating General Apps through the Marketplace API. They are based on
live API behavior observed on 2026-07-08 and should be treated as more precise than the
high-level endpoint inventory.

### General App Manifest Modes

For General App OAuth manifests, `oauth_information.usage` currently accepts only:

| Usage value | Meaning | Scope shape |
|-------------|---------|-------------|
| `USER_OPERATION` | User-level General App | User scopes such as `user:read:user` |
| `ADMIN_MANAGEMENT` | Admin/account-level General App | Admin scopes such as `user:read:user:admin` |

Invalid examples:
- `ACCOUNT_OPERATION` is rejected.
- A `USER_OPERATION` manifest with an admin scope is rejected.
- An `ADMIN_MANAGEMENT` manifest with a user scope is rejected.
- App-owned Marketplace scopes such as `marketplace:write:event_subscription` are rejected in `oauth_information.scopes`; those belong to `client_credentials` token usage, not the authorization-code OAuth manifest scope list.
- If scopes are present, at least one scope must be non-optional.
- Invalid scope names return `ok: false` from manifest validation.
- User-level scopes can be granular names such as `user:read:email`.
- Admin/account-level scopes can be granular names such as `meeting:read:meeting:admin`.

Minimal user-level manifest:

```json
{
  "display_information": {
    "display_name": "My User General App"
  },
  "oauth_information": {
    "usage": "USER_OPERATION",
    "development_redirect_uri": "https://example.com/oauth/callback",
    "production_redirect_uri": "https://example.com/oauth/callback",
    "oauth_allow_list": ["https://example.com/oauth/callback"],
    "scopes": [
      { "scope": "user:read:user", "optional": false }
    ],
    "strict_mode": false,
    "subdomain_strict_mode": false
  },
  "developer_information": {
    "developer_name": "Example Developer",
    "developer_email": "developer@example.com",
    "company_name": "Example Co",
    "website": "https://example.com",
    "privacy_policy": "https://example.com/privacy",
    "terms_of_use": "https://example.com/terms",
    "support": "https://example.com/support"
  },
  "features": {
    "products": []
  }
}
```

Minimal admin/account-level manifest:

```json
{
  "display_information": {
    "display_name": "My Admin General App"
  },
  "oauth_information": {
    "usage": "ADMIN_MANAGEMENT",
    "development_redirect_uri": "https://example.com/oauth/callback",
    "production_redirect_uri": "https://example.com/oauth/callback",
    "oauth_allow_list": ["https://example.com/oauth/callback"],
    "scopes": [
      { "scope": "user:read:user:admin", "optional": false }
    ],
    "strict_mode": false,
    "subdomain_strict_mode": false
  },
  "developer_information": {
    "developer_name": "Example Developer",
    "developer_email": "developer@example.com",
    "company_name": "Example Co",
    "website": "https://example.com",
    "privacy_policy": "https://example.com/privacy",
    "terms_of_use": "https://example.com/terms",
    "support": "https://example.com/support"
  },
  "features": {
    "products": []
  }
}
```

### Manifest Validation Response Shape

`POST /marketplace/apps/manifest/validate` returns HTTP `200` for both valid and invalid manifests.
Always inspect `ok`.

Valid response:

```json
{
  "ok": true,
  "error": "",
  "errors": null
}
```

Invalid response:

```json
{
  "ok": false,
  "error": "invalid_manifest",
  "errors": [
    {
      "setting": "/oauth_information/scopes[0]",
      "message": "user:read:user:admin is not a valid scope name when usage is USER_OPERATION"
    }
  ]
}
```

Common validation quirks:
- Empty `oauth_information.scopes: []` can validate, but an installable production app usually needs scopes aligned to the intended API calls.
- Missing `oauth_information.scopes` can also validate and can create an app with `app_scopes: []`; do not use this for apps that need API access.
- If `oauth_information.scopes` is non-empty, include at least one `{ "optional": false }` scope.
- A scope entry missing the `optional` field, using `"optional": "false"` as a string, or duplicating the same scope can validate. Do not rely on that permissiveness; emit boolean `optional` values and dedupe scopes yourself.
- Classic scope names such as `user:read` can be rejected for current General App manifests; prefer granular names such as `user:read:email`.
- Chatbot scope `imchat:bot` validated under `ADMIN_MANAGEMENT` with `products: ["ZOOM_CHAT"]`, but was rejected under `USER_OPERATION` in live tests.
- If `features.products` is empty, do not set `features.development_home_uri` or `features.domain_allow_list`.
- `features.domain_allow_list` does not accept plain strings. When a selected product requires domains, use object entries such as `{ "domain": "example.com" }`.
- `display_information.display_name` is required and must be less than 50 characters.
- If `features.team_chat_subscription.enable` is `true`, include `features.team_chat_subscription.slash_command.development_message_url`. Missing or blank development message URL fails validation even when `production_message_url` is present. `production_message_url` can be omitted or blank for development-only chatbot manifests.
- Team Chat shortcut objects use `action_types` as an array. Do not use singular `action_type`; invalid `action_types` enum values are rejected.
- Event subscription manifests should use `WEB_SOCKET` when selecting a WebSocket subscription type. Do not invent lowercase or alternate enum spellings.
- `features.sdk_domain_allow_list` can be schema-valid but account/feature gated. A live validation run reported `ok: false` with a message equivalent to `SDK domain allowlist feature is not enabled for this account`.
- The validation endpoint is not a complete production-readiness check. It accepted invalid-looking development redirect URLs, missing redirect fields, missing developer information, missing `features`, and unknown top-level keys in live tests on 2026-07-08.
- Manifest validation can pass feature sections independently; publish/review may still require additional Basic Information, privacy, support, domain, scope justification, event, and product-specific configuration that validation does not enforce.
- Existing saved manifests are not guaranteed to round-trip through current validation. In live app exports, saved RTMS meeting/webinar event subscriptions and duplicate slash command names could export successfully but fail `POST /manifest/validate` when submitted again.

### Broad Manifest Schema Coverage

Official manifest schema pages currently document these top-level General App sections:

| Section | Officially documented fields or values | Validation guidance |
|---------|----------------------------------------|---------------------|
| `display_information` | `display_name`, `description`, `long_description`, `company_name`, `discover_type`, `install_type`, `marketplace_categories`, `industry_vertical`, `market_segments`, `business_lines`, `privacy_url`, `terms_url`, `support_url`, `doc_url`, `config_url` | Use documented enum values such as `UNLISTED` / `LISTED` and `APP_DIRECTORY` / `LANDING_PAGE`; invalid enum values were rejected in validation probes |
| `oauth_information` | `usage`, redirect URIs, allow-list, strict modes, `scopes`, `scope_description`, `enable_private_pkce_client`, `enable_public_pkce_client`, public key settings | Use `USER_OPERATION` or `ADMIN_MANAGEMENT`; lowercase variants were rejected |
| `oauth_information.*_public_key` | `enabled`, `well_known_url`, `keys[]` JWK fields for RSA or EC keys | `well_known_url` is mutually exclusive with inline `keys`; inline RSA and EC JWK shapes are documented by Zoom and were accepted in a parallel validation run |
| `features.products` | `ZOOM_MEETING`, `ZOOM_CHAT`, `ZOOM_PHONE`, `ZOOM_ROOM`, `ZOOM_WEBINAR`, `ZOOM_CONTACT_CENTER`, `ZOOM_WHITEBOARD`, `ZOOM_EVENTS` | Invalid product enum values were rejected |
| `features` | `connector`, `in_client_feature`, `zoom_client_support`, `embed`, `team_chat_subscription`, `event_subscription`, `connect`, `customer_form`, `marketplace_actions`, `sdk_domain_allow_list`, `plugin_sdk` | Prefer the official key `customer_form`. A parallel validation run also saw `custom_form` accepted, but do not emit it in new manifests |
| Template feature sections | `in_client_feature`, `zoom_client_support`, `embed`, `team_chat_subscription`, `event_subscription` | These can validate independently; Marketplace review can still require product-specific evidence and complete production URLs |
| Schema-only feature sections | `connector`, `connect`, `customer_form`, `marketplace_actions`, `marketplace_triggers`, `plugin_sdk` | Parallel validation reported these accepted with `enable: true`, but treat them as schema coverage only unless the app actually uses the feature |

Broad combined manifests are useful for schema testing but are not yet proven as one-shot
create/persist inputs. A parallel live run reported that a broad combined manifest validated
across many sections, but `POST /marketplace/apps` returned `401 Access Denied` for the
full combined create attempt. When using broad manifests, validate first, then create/update
with the smallest feature set required by the app, and confirm persistence with
`GET /marketplace/apps/{appId}/manifest`.

Official schema sources:
- Manifest schema overview: https://developers.zoom.us/docs/build-flow/manifests/schema/
- Manifest template: https://developers.zoom.us/docs/build-flow/manifests/schema/manifest-template/
- Display information: https://developers.zoom.us/docs/build-flow/manifests/schema/display-information/
- OAuth information: https://developers.zoom.us/docs/build-flow/manifests/schema/oauth-information/
- Features: https://developers.zoom.us/docs/build-flow/manifests/schema/features/
- Marketplace app APIs: https://developers.zoom.us/docs/api/marketplace/
- Marketplace master app APIs: https://developers.zoom.us/docs/api/marketplace/ma/
- Marketplace API inventory: https://developers.zoom.us/api-hub/marketplace/methods/endpoints.json

For reusable request bodies, use [Marketplace App Templates](marketplace-app-templates.md).
Select the closest scenario, replace every `example.com` value, reduce scopes to the exact
operations required, validate General App manifests, and then create the app.

### General App Create/Get/Manifest/Update/Delete Shapes

Create General App:

```http
POST /v2/marketplace/apps
```

Request:

```json
{
  "app_type": "general",
  "app_name": "My General App",
  "contact_name": "Example Developer",
  "contact_email": "developer@example.com",
  "company_name": "Example Co",
  "manifest": {
    "...": "valid manifest"
  }
}
```

Observed minimal create response:

```json
{
  "app_id": "n4C4AvnpSm6Jg9JSV0bgww"
}
```

Possible create response with credentials:

```json
{
  "app_id": "n4C4AvnpSm6Jg9JSV0bgww",
  "app_name": "My General App",
  "app_type": "general",
  "development_credentials": {
    "client_id": "CLIENT_ID",
    "client_secret": "CLIENT_SECRET"
  }
}
```

Observed create response with a `credentials` wrapper:

```json
{
  "app_id": "n4C4AvnpSm6Jg9JSV0bgww",
  "oauth_authorize_url": "https://zoom.us/oauth/authorize?...",
  "credentials": {
    "client_id": "CLIENT_ID",
    "client_secret": "CLIENT_SECRET"
  }
}
```

Do not rely on documented `201` or a single fixed response shape for General App creates.
Live behavior can return HTTP `200`; it may return only `app_id` or may include generated
General App credentials. If credentials are returned, those Client ID/Secret values can be
used to mint `client_credentials` tokens for app-owned scopes when the app has the required
app-owned scopes. If the response only returns `app_id`, retrieve credentials from the
Marketplace app credentials surface or a documented credential endpoint before testing
app-owned token flows. Credential fields have appeared as `credentials` and
`development_credentials`; treat both as possible shapes and never log the secret values.

Get created app:

```http
GET /v2/marketplace/apps/{appId}
```

Observed user-level response shape:

```json
{
  "app_name": "My General App",
  "app_type": "OAuthApp",
  "app_status": "unpublished",
  "app_scopes": ["user:read:user"]
}
```

Observed admin/account-level response shape:

```json
{
  "app_name": "My Admin General App",
  "app_type": "OAuthApp",
  "app_status": "unpublished",
  "app_scopes": ["user:read:user:admin"]
}
```

Export manifest:

```http
GET /v2/marketplace/apps/{appId}/manifest
```

Response shape:

```json
{
  "manifest": {
    "...": "inner manifest"
  }
}
```

### Manifest Export Caveats

Export caveats observed from existing apps:
- Draft General apps with granular scopes exported cleanly in live read-only checks.
- Published apps and some legacy/non-General/S2S-style apps can return HTTP `404` with Zoom `code: 1500`, for example `You can only export Draft General app's manifest now`.
- Server-to-Server OAuth/private OAuth-style apps do not export through this endpoint as General App manifests; expect `You can only export Draft General app's manifest now`.
- Older or non-granular General apps may return `You can only export manifest with app support granular scope now`; treat this as an exportability limitation, not proof the app is misconfigured.
- `GET /manifest` can export a saved manifest that fails current `POST /manifest/validate`; always validate before using an exported manifest as an update template.
- A saved Team Chat chatbot slash command can fail validation on export/re-submit if the command name is already in use in the account. Rename slash commands when cloning manifests.

Update manifest:

```http
PUT /v2/marketplace/apps/{appId}/manifest
```

Request shape:

```json
{
  "manifest": {
    "...": "inner manifest only"
  }
}
```

Do not pass the full export response as the `manifest` value. This is wrong:

```json
{
  "manifest": {
    "manifest": {
      "...": "nested wrapper"
    }
  }
}
```

Passing the export wrapper instead of the inner manifest can produce misleading validation errors.
Successful `PUT /manifest` can return HTTP `200` with an empty body.

Delete created app:

```http
DELETE /v2/marketplace/apps/{appId}
```

Observed delete response is HTTP `200` with an empty body. Do not require the documented
`204` to treat deletion as successful.

### Permissive Manifest Cases Observed in Create Tests

Live create/delete tests on 2026-07-08 confirmed that some manifests accepted by
`/manifest/validate` are also accepted by `POST /marketplace/apps`. Treat these as
Marketplace leniency, not recommended manifest style:

| Case | Create result | Export/get behavior |
|------|---------------|---------------------|
| Empty `oauth_information.scopes: []` | HTTP `200` | `GET /apps/{appId}` returned `app_scopes: []`; exported manifest omitted/nullified scopes |
| Missing `oauth_information.scopes` | HTTP `200` | `app_scopes: []`; exported manifest omitted/nullified scopes |
| `oauth_allow_list` that does not include redirect URI | HTTP `200` | Export preserved mismatched allow-list |
| Missing `development_redirect_uri` | HTTP `200` | Exported `development_redirect_uri: ""` |
| Invalid-looking `development_redirect_uri: "not-a-url"` | HTTP `200` | Export preserved the invalid-looking string |
| Missing `developer_information` | HTTP `200` | Export only included `display_information`, `oauth_information`, and `features` |
| Missing `features` | HTTP `200` | Export reintroduced `features` |
| Unknown top-level key | HTTP `200` | Export dropped the unknown key |

Practical rule: generate strict manifests even if the API accepts loose ones. Include scopes,
valid HTTPS redirect URIs, a matching allow-list, developer information, and `features.products`
explicitly. Use validation as a syntax/scope-mode check, then fetch the manifest after create or
update to confirm what Zoom actually persisted.

### Existing App Manifest Archetypes Observed

Read-only exports of existing apps on 2026-07-08 surfaced these useful shape patterns:

| Archetype | Manifest shape | Route to |
|-----------|----------------|----------|
| Meeting SDK / meeting-surface app | Usually `products: ["ZOOM_MEETING"]`, Meeting-related scopes, optional embed/in-client feature sections, event subscriptions only when the app also listens for meeting events | [Meeting SDK](../../meeting-sdk/SKILL.md) or [Meeting SDK Web](../../meeting-sdk/web/SKILL.md) |
| RTMS meeting/webinar app | `products: ["ZOOM_MEETING"]`, `event_subscription.enable: true`, RTMS event names may appear in saved exports but fail current manifest validation | [RTMS](../../rtms/SKILL.md) and [RTMS webhooks](../../rtms/references/webhooks.md) |
| ZCC/Phone surface app | `products` can include combinations such as `["ZOOM_CONTACT_CENTER", "ZOOM_PHONE"]`; may have zero OAuth scopes when the app is surface/configuration focused | [Contact Center](../../contact-center/SKILL.md) and [Phone](../../phone/SKILL.md) |
| Team Chat user API app | `usage: "USER_OPERATION"`, `products: ["ZOOM_CHAT"]`, Team Chat user scopes such as `team_chat:*`, optional `imchat:userapp`, `team_chat_subscription.enable: false` | [Team Chat](../../team-chat/SKILL.md) |
| Admin OAuth / account-scoped API app | `usage: "ADMIN_MANAGEMENT"`, admin granular scopes, often `products: []` or a product-specific surface, admin install required | [OAuth](../../oauth/SKILL.md) and [app types](../../general/references/app-types.md) |
| Chatbot app | `usage: "ADMIN_MANAGEMENT"`, `products: ["ZOOM_CHAT"]`, scope `imchat:bot`, `team_chat_subscription.enable: true`, slash command development message URL present, production message URL may be blank | [Team Chat chatbot](../../team-chat/SKILL.md) |
| Webhook-heavy app | `event_subscription.enable: true` can appear even when `products: []`; event entries use `event_usage` such as `EVENT_FOR_USER` or `EVENT_FOR_ADMIN`; large event lists may not round-trip through validation unchanged | [Webhooks](../../webhooks/SKILL.md) and this Marketplace reference |
| WebSocket event app | Account-level only; use S2S or admin-managed General App, then select WebSocket delivery in Features > Access because the public manifest schema does not expose a reliable one-shot field | [WebSockets](../../websockets/SKILL.md) |
| Plugin SDK companion | `usage: "USER_OPERATION"`, scope `plugin_sdk:read:connection_meta`, PKCE for native clients, and `plugin_sdk.enable: true`; the OAuth user must match the user signed in to Zoom Workplace | [Plugin SDK](../../plugin-sdk/SKILL.md) |
| Claude MCP Connector / MCP API app | `usage: "USER_OPERATION"`, `products: []` or omitted, no Team Chat subscription, scopes for AI Companion search, docs import/export, meeting assets/search, recordings, or Team Chat MCP workflows | [Zoom MCP](../../zoom-mcp/SKILL.md) and [REST API](../SKILL.md) |

Do not infer app behavior from `app_type` alone. Existing apps returned both `OAuthApp` and
`ZoomApp` for General-app-style manifests; inspect `oauth_information.usage`, `products`,
features, scopes, and event subscriptions instead.

Cross-reference this section from:
- [General app types](../../general/references/app-types.md) when choosing General App scope level and surfaces.
- [Marketplace portal guide](../../general/references/marketplace.md) when exporting or cloning manifests.
- [Marketplace publishing use case](../../general/use-cases/marketplace-publishing.md) when preparing review-ready app configurations.

### RTMS Event Subscription Validation Drift

Existing saved RTMS apps exported event subscriptions containing names such as:

```json
[
  "meeting.rtms_started",
  "meeting.rtms_stopped",
  "meeting.rtms_interrupted",
  "webinar.rtms_started",
  "webinar.rtms_stopped",
  "webinar.rtms_interrupted",
  "rtms.concurrency_limited",
  "rtms.concurrency_near_limit"
]
```

However, a live validation probe on 2026-07-08 rejected those meeting/webinar RTMS event names
inside `features.event_subscription.events[].event_types`. The same probe accepted Contact
Center Voice RTMS event names:

```json
[
  "contact_center.voice_rtms_started",
  "contact_center.voice_rtms_stopped",
  "contact_center.voice_rtms_interrupted"
]
```

Follow-up create/update probe:
- `POST /manifest/validate` rejected `meeting.rtms_*` and `rtms.concurrency_*` event names.
- `POST /marketplace/apps` with those event names also returned `400 invalid_manifest`.
- `PUT /marketplace/apps/{appId}/manifest` against an existing draft General App returned
  HTTP `200` and a subsequent `GET /manifest` showed those RTMS event names persisted.

Practical rule: for RTMS meeting/webinar event subscriptions, do not use create-time manifest
validation as the only source of truth. Prefer Marketplace UI or the event-subscription API.
If you must use manifests, create the draft app with a valid baseline manifest, then test
`PUT /manifest` and immediately `GET /manifest` to confirm the RTMS event subscriptions
actually persisted. Do not assume every validation failure can be bypassed this way; most
structural validation failures still blocked both create and update in live tests.

### Event Subscription Pairing

Event subscription APIs are app-owned operations and should use a General App
`client_credentials` token when the subscription scopes require app-owned access.

Observed working user-managed pairing from a parallel validation run:

```json
{
  "subscription_scope": "user",
  "user_ids": ["USER_ID"]
}
```

This worked with a user-managed General App and a `client_credentials` token minted from
that app's Client ID and Client Secret. A direct verification run on 2026-07-08 could not
reproduce the token step because `POST /marketplace/apps` returned only `app_id`, so treat
the subscription pairing as dependent on having access to the generated General App
credentials first.

Observed admin/account-level caveat:
- Creating an account-scoped event subscription for an admin-managed General App before the app is authorized/installed can fail with:

```text
The user don't authorize the app.
```

Treat account-scoped event subscription setup as requiring the General App to be authorized/installed first.

### Webhook-only and S2S Creation Quirks

The public app-creation endpoint accepts only the documented `POST /marketplace/apps` shape, but
live behavior is nuanced:

- `app_type: "general"` with a valid manifest creates a General/OAuth-style app.
- `app_type: "webhook_only"` with the same valid manifest was accepted, but `GET /marketplace/apps/{appId}` reported `app_type: "OAuthApp"` and OAuth scopes. Do not treat this as proof that the API created a true Webhook-only App.
- `app_type: "webhook"` behaved similarly in live probing.

For Server-to-Server OAuth creation, the current Marketplace API description lists
`app_type: "s2s_oauth"` on both create surfaces, but live routing on 2026-07-10 required the
account-scoped endpoint:

- `POST /v2/marketplace/apps` returned HTTP `404`, Zoom code `1500`, directing the caller to
  `/v2/accounts/{accountId}/marketplace/apps` for Meeting SDK or S2S OAuth app creation.
- `POST /v2/accounts/{accountId}/marketplace/apps` requires
  `marketplace:write:app:master` and is account-owner/master-scope sensitive.
- Set `active: true` only when `scopes` contains at least one valid scope. Use
  `active: false` to create an inactive draft.
- Do not send a General App `manifest` for `s2s_oauth`; send top-level `scopes`.
- Successful responses can include generated development and production credentials.
  Capture them once and never log or commit client secrets.

A live authorization probe on 2026-07-10 reached the account-scoped endpoint, but the supplied
tokens lacked its required master scope:

```json
{
  "code": 201,
  "message": "Invalid access token, does not contain scopes:[marketplace:write:app:master]."
}
```

No app was created in the probes. Use the account-scoped route and a token containing
`marketplace:write:app:master`; do not rely on the broader operation description for the regular
create endpoint until runtime behavior changes.

## Coverage

| Metric | Value |
|--------|-------|
| Endpoint operations | 28 |
| Path templates | 19 |
| Tags | 3 |

## Tag Index

| Tag | Operations |
|-----|------------|
| App | 24 |
| Apps | 1 |
| Manifest | 3 |

## Endpoints by Tag

### App

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/app/notifications` | Send app notifications | `Sendappnotifications` |
| GET | `/marketplace/app/custom_fields` | Get user's custom field values | `getCustomFieldValues` |
| DELETE | `/marketplace/app/event_subscription` | Unsubscribe app event subscription | `unSubscribeEventSubscription` |
| GET | `/marketplace/app/event_subscription` | Get user or account event subscription | `getUserEventSubscriptions` |
| PATCH | `/marketplace/app/event_subscription` | Update an event subscription | `updateEventSubscription` |
| POST | `/marketplace/app/event_subscription` | Create an event subscription | `createEventSubscription` |
| DELETE | `/marketplace/app/event_subscription/{eventSubscriptionId}` | Delete an event subscription | `deleteEventSubscription` |
| PATCH | `/marketplace/app/event_subscription/{eventSubscriptionId}` | Subscribe an event subscription | `subscribeEventSubscription` |
| GET | `/marketplace/apps` | List apps | `ListApps` |
| POST | `/marketplace/apps` | Create apps | `CreateApps` |
| DELETE | `/marketplace/apps/{appId}` | Deletes an app | `deleteApp` |
| GET | `/marketplace/apps/{appId}` | Get information about an app | `getAppInfo` |
| GET | `/marketplace/apps/{appId}/api_call_logs` | Get API call logs | `Getapicalllogs` |
| POST | `/marketplace/apps/{appId}/deeplink` | Generate Zoom App Deeplink | `GenerateZoomAppDeeplink` |
| POST | `/marketplace/apps/{appId}/preApprove` | Update app pre approval setting | `Updateapppreapprovalsetting` |
| GET | `/marketplace/apps/{appId}/requests` | Get an app's user requests | `getAppUserRequests` |
| PATCH | `/marketplace/apps/{appId}/requests` | Update app's request status | `updateAppRequestStatus` |
| POST | `/marketplace/apps/{appId}/requests` | Add app allow requests for users | `AddAppAllowRequestsForUsers` |
| POST | `/marketplace/apps/{appId}/rotate_client_secret` | Rotate client secret | `RotateClientSecret` |
| GET | `/marketplace/apps/{appId}/webhook_logs` | Get webhook logs | `getWebhookLogs` |
| GET | `/marketplace/monetization/entitlements` | Get app user entitlements | `getAppUserEntitlementRequests` |
| GET | `/marketplace/users/{userId}/apps` | Get a user's app requests | `getUserAppRequests` |
| PATCH | `/marketplace/users/{userId}/apps/{appId}/subscription` | Enable or disable user app subscription | `Enable/Disableuserappsubscription` |
| GET | `/marketplace/users/{userId}/entitlements` | Get a user's entitlements | `getUserEntitlementRequests` |

### Apps

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/zoomapp/deeplink` | Generate an app deeplink | `generateAppDeeplink` |

### Manifest

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/marketplace/apps/manifest/validate` | Validate an app manifest | `validatingManifest` |
| GET | `/marketplace/apps/{appId}/manifest` | Export an app manifest from an existing app | `getAppManifest` |
| PUT | `/marketplace/apps/{appId}/manifest` | Update an app by manifest | `updateAppByManifest` |
