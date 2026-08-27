# Marketplace Connect, Actions, and Triggers

Use this reference when a General App manifest must expose an external REST API or MCP server
inside Zoom, or when it must define manifest-managed actions and triggers. These capabilities are
feature blocks merged into a complete General App manifest; they are not standalone app-create
requests.

## Do Not Confuse the Two MCP Directions

| Goal | Configuration |
|------|---------------|
| Connect an AI client to a Zoom-hosted MCP server | Create a user-managed General App with the exact Zoom MCP tool scopes and PKCE. Use the `mcp-*` templates in the [template selector](marketplace-app-templates.md). |
| Expose a third-party MCP server to supported Zoom surfaces | Add `features.connect.mcp` to a General App. Use the external MCP fragment below. |

For `features.connect.mcp`, Zoom is the MCP client. Zoom obtains the OAuth client ID through
Dynamic Client Registration (DCR) or a Client ID Metadata Document (CIMD). Do not add a static
`security.oauth_config.client_id`; manifest import rejects it for this mode.

## Feature Fragments

Merge only the required fragment into the `features` object of a complete exported or selected
General App manifest:

| Capability | Fragment |
|------------|----------|
| External REST API routes | [Connect REST fragment](../assets/marketplace-apps/marketplace-manifest-fragment-for-connect-rest-api.json) |
| External MCP server | [Connect external MCP fragment](../assets/marketplace-apps/marketplace-manifest-fragment-for-connect-external-mcp.json) |
| Custom AI Agent action | [Custom action fragment](../assets/marketplace-apps/marketplace-manifest-fragment-for-custom-action.json) |
| Zoom Phone built-in trigger | [Zoom Phone trigger fragment](../assets/marketplace-apps/marketplace-manifest-fragment-for-trigger-zoom-phone.json) |
| ZCC Voice Bot built-in trigger | [ZCC Voice Bot trigger fragment](../assets/marketplace-apps/marketplace-manifest-fragment-for-trigger-zcc-voice-bot.json) |

Replace all `example.com` URLs, descriptions, keys, schemas, and mappings. Validate the complete
manifest and read it back after update. Never submit a fragment by itself.

## Connect

`features.connect` can define a base REST URL, authentication, up to 100 routes, incoming
webhooks, and an optional MCP server.

Supported `security.type` values:

| Value | Authentication |
|-------|----------------|
| `1` | API key |
| `5` | Basic authentication |
| `6` | Bearer token |
| `7` | JWT bearer |
| `8` | No authentication |
| `9` | AWS Signature V4 |
| `31` | OAuth 2.0 authorization code |
| `33` | OAuth 2.0 password |
| `34` | OAuth 2.0 client credentials |
| `36` | OAuth 2.0 authorization code with PKCE |

Important rules:

- For authorization code types `31` and `36`, Zoom assigns the redirect URL after import. Do
  not place a redirect URL in the Connect manifest; register the assigned URL with the external
  authorization server.
- `token`, `password`, `secret`, and `client_secret` are write-only. They are accepted on import
  but omitted from exports. Preserve them outside the exported manifest and re-supply them only
  when the update requires changing that security configuration.
- Every route needs a stable, unique `key`; paths are appended to `connect.url`.
- Incoming webhook `key` values must be unique. A `RestHook` must reference existing subscribe
  and unsubscribe route keys; a `Static` webhook must not contain `rest_hook`.
- When `mcp` exists, it overrides `security`. `mcp.base_url` must be HTTPS and `mcp.ext` must be
  present, even when it is `{}`.

## Custom Actions

Use `features.marketplace_actions` for developer-defined actions. The currently documented
custom-action location is `VIRTUAL_AGENT`.

Each action must use exactly one wiring mode:

| Mode | Include | Exclude |
|------|---------|---------|
| Connect endpoint | `endpoint_key`, `input_mapping`, optional `output_mapping` | `script` |
| Inline script | `script` | `endpoint_key`, `input_mapping`, `output_mapping` |

Keep `command_id` unique and stable. For new actions use `definition_type: 2` and JSON Schema
input/output definitions. The `endpoint_key` must match a route key in `features.connect.routes`.
Do not add `built_in_config` to custom actions unless Zoom publishes a built-in action contract.

## Built-In Triggers

The public manifest schema currently documents these built-in trigger definitions:

| Product | `name` | `command_id` | `definition_id` | Version |
|---------|--------|--------------|-----------------|---------|
| Zoom Phone | `Zoom Phone All Events` | `zoom_phone_all_events` | `ZoomPhoneAllEvents2025_Consolidated` | `1.0.0` |
| Zoom Contact Center Voice Bot | `Voice Bot All Events` | `voicebot_all_events` | `VoiceBotAllEvents2025_Consolidated` | `1.0.0` |

Use those values exactly. For each `static_webhook` trigger, provide
`static_webhook_config.connector_webhook_key` matching the app's Connect webhook key and keep it
at 128 characters or fewer. Do not include `locations`, `output_definition`, `script`,
`global_webhook_config`, or `resthook_config`; Zoom supplies the built-in contract.

## Schema Drift Rules

- Preserve the key returned by manifest export. Current official pages have used both
  `features.customer_form` and `features.custom_form`; do not rename either during an unrelated
  update.
- For a new app that needs this form feature, start from the latest official manifest template,
  use its current key, and require `POST /manifest/validate` to return `ok: true`.
- Validation proves schema acceptance, not product entitlement, persistence, installation, or
  Marketplace review readiness.
- Export the complete manifest before editing, preserve unknown fields, and read back after every
  update. Follow the [manifest update workflow](marketplace-manifest-update-workflow.md).

## Official References

- Connect: https://developers.zoom.us/docs/build-flow/manifests/schema/connect/
- Actions: https://developers.zoom.us/docs/build-flow/manifests/schema/actions/
- Triggers: https://developers.zoom.us/docs/build-flow/manifests/schema/trigger/
- Manifest template: https://developers.zoom.us/docs/build-flow/manifests/schema/manifest-template/
