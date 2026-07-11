# Zoom App Marketplace

Navigate the Zoom Marketplace developer portal.

## Overview

The [Zoom App Marketplace](https://marketplace.zoom.us/) is where you create, configure, and publish Zoom apps.

## Getting Started

1. Go to [marketplace.zoom.us](https://marketplace.zoom.us/)
2. Sign in with your Zoom account
3. Click **Develop** → **Build App**
4. Choose app type
5. Configure app settings

## Portal Sections

### Develop

- **Build App** - Create new apps
- **Manage** - Edit existing apps
- **Logs** - View API and webhook logs

### App Configuration

| Section | Purpose |
|---------|---------|
| **App Credentials** | SDK Key/Secret, Client ID/Secret |
| **Scopes** | Configure OAuth permissions |
| **Feature** | Enable Meeting SDK, Video SDK, Webhooks |
| **Activation** | Make app installable |

## Manifest Export and Archetypes

Use [Marketplace App Management](../../rest-api/references/marketplace-apps.md) for
manifest API behavior, export caveats, and observed archetypes.

Key cross-product rules:
- Draft General apps with granular scopes export cleanly.
- Server-to-Server OAuth/private OAuth-style apps do not export as General App manifests.
- Older or non-granular General apps may fail export with `You can only export manifest with app support granular scope now`.
- Use the [Existing App Manifest Archetypes](../../rest-api/references/marketplace-apps.md#existing-app-manifest-archetypes-observed) table when cloning or designing Meeting SDK, RTMS, ZCC/Phone, Team Chat, Admin OAuth, Chatbot, webhook-heavy, or MCP connector configurations.

## SDK Downloads

**Important:** Meeting SDK and Video SDK must be downloaded from Marketplace after signing in. They are not available on public package managers (except Web SDKs via npm).

1. Go to your app's **Download** section
2. Select platform (iOS, Android, Windows, macOS, Linux)
3. Download SDK package

## Credentials

### General Apps

- **Client ID** - Public identifier
- **Client Secret** - Keep secret, server-side only
- Use these same credentials to request `client_credentials` tokens when a
  General App endpoint requires app-owned scopes.

### Meeting SDK Apps

- **Client ID** - Used as the Meeting SDK JWT `appKey`
- **Client Secret** - Used to sign the Meeting SDK JWT; keep server-side
- Legacy Meeting SDK Key/Secret migration was enforced on June 27, 2026.

### Build Platform SDK Apps

Video SDK, Cobrowse, and AI Services use the credentials exposed by their Build Platform app.
Credential labels and token contracts are product-specific; use the corresponding product skill
instead of assuming Meeting SDK Client ID/Secret semantics.

## Publishing

To publish to Marketplace:

1. Complete app configuration
2. Submit for review
3. Address feedback
4. Get approved
5. Go live

## Resources

- **Marketplace**: https://marketplace.zoom.us/
- **Developer docs**: https://developers.zoom.us/
