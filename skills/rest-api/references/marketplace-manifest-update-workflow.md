# Marketplace Manifest Update Workflow

Use this workflow when an MCP client or automation service must change the settings of an
existing Zoom Marketplace General App. A manifest update is a full replacement, not a JSON
patch.

## Supported Boundary

This workflow is for existing **General Apps** whose configuration is represented by the General
App manifest. Do not use it for `s2s_oauth` or `meeting_sdk` apps; those use native creation and
credential/configuration fields instead.

## Required Permissions

Use the least-privileged Marketplace scope that can read and update the target app:

| Operation | User-level app | Admin-level app |
|-----------|----------------|-----------------|
| Export manifest | `marketplace:read:app` | `marketplace:read:app:admin` |
| Validate manifest | `marketplace:read:app` | `marketplace:read:app:admin` |
| Update manifest | `marketplace:write:app` | `marketplace:write:app:admin` |

Confirm the caller owns or has permission to edit the target app. A token that can create an app
is not automatically authorized to update every app in the account.

## Safe Sequence

1. Identify the exact `app_id`. Never select an app by display name alone.
2. Export the current manifest with `GET /v2/marketplace/apps/{appId}/manifest`.
3. Keep the returned `manifest` object and discard only the outer response wrapper.
4. Apply the requested changes to the complete manifest in memory.
5. Preserve all unmodified fields, including URLs, scopes, products, feature settings, and
   subscription configuration.
6. Validate the complete candidate with `POST /v2/marketplace/apps/manifest/validate`.
7. Require both a successful HTTP response and `ok: true`.
8. Update with `PUT /v2/marketplace/apps/{appId}/manifest`.
9. Require the documented `204` success response, while tolerating an observed successful empty
   response if the API returns one.
10. Export the manifest again and compare the resulting configuration with the intended one.

## Request Shapes

Export response:

```json
{
  "manifest": {
    "display_information": {},
    "oauth_information": {},
    "features": {}
  }
}
```

Validation request without an app context:

```json
{
  "manifest": {
    "...complete manifest...": "..."
  }
}
```

Validation request for an existing app:

```json
{
  "app_id": "existing-app-id",
  "manifest": {
    "...complete manifest...": "..."
  }
}
```

Update request:

```json
{
  "manifest": {
    "...complete manifest...": "..."
  }
}
```

Do not double-wrap the exported object:

```json
{
  "manifest": {
    "manifest": {
      "...incorrect nested manifest...": "..."
    }
  }
}
```

## MCP Update Rules

When updating an MCP app:

- Export the existing app before changing scopes or PKCE settings.
- Keep `usage: "USER_OPERATION"` for the currently supported hosted MCP user flows.
- Keep the MCP client's exact redirect URI and OAuth allow list.
- Add only the scopes required by the selected MCP server tools.
- Do not add app-owned Marketplace event or WebSocket scopes to `oauth_information.scopes`.
- Reauthorize the app after a scope change so newly issued access tokens contain the new scopes.
- Run `tools/list` after authorization to verify that the active MCP server exposes the expected
  tool surface.

## Failure Handling

- If validation returns HTTP `200` with `ok: false`, do not call `PUT`.
- If validation rejects a feature because it is account-gated, keep the current manifest and
  report the feature gate instead of silently removing the feature.
- If `PUT` fails, retain the exported pre-change manifest for recovery and retry only after fixing
  the reported field.
- If `PUT` succeeds but the read-back differs, treat the update as normalized or incomplete and
  compare field-by-field.
- Never remove existing scopes or event subscriptions unless the user explicitly requested their
  removal.

## Official Endpoints

- Validate: `POST https://api.zoom.us/v2/marketplace/apps/manifest/validate`
- Export: `GET https://api.zoom.us/v2/marketplace/apps/{appId}/manifest`
- Update: `PUT https://api.zoom.us/v2/marketplace/apps/{appId}/manifest`
- API reference: https://developers.zoom.us/docs/api/marketplace/
- Manifest overview: https://developers.zoom.us/docs/build-flow/manifests/
