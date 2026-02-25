# Marketplace Apps API

Manage Zoom Marketplace app listings, versions, and distribution.

## Overview

The Marketplace Apps API enables developers to programmatically manage their Zoom App Marketplace listings, submit app versions, track review status, and manage app distribution settings.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with marketplace scopes:
- `marketplace:read` - Read app data
- `marketplace:write` - Manage app listings

## Key Endpoints

### App Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/marketplace/apps` | List your apps |
| GET | `/marketplace/apps/{appId}` | Get app details |
| PATCH | `/marketplace/apps/{appId}` | Update app |
| DELETE | `/marketplace/apps/{appId}` | Delete app (draft only) |

### App Versions

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/marketplace/apps/{appId}/versions` | List versions |
| POST | `/marketplace/apps/{appId}/versions` | Create version |
| GET | `/marketplace/apps/{appId}/versions/{versionId}` | Get version |
| PATCH | `/marketplace/apps/{appId}/versions/{versionId}` | Update version |
| POST | `/marketplace/apps/{appId}/versions/{versionId}/submit` | Submit for review |

### Review Status

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/marketplace/apps/{appId}/review` | Get review status |
| GET | `/marketplace/apps/{appId}/review/feedback` | Get review feedback |

### Publishing

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/marketplace/apps/{appId}/publish` | Publish app |
| POST | `/marketplace/apps/{appId}/unpublish` | Unpublish app |
| POST | `/marketplace/apps/{appId}/deprecate` | Deprecate app |

### Analytics

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/marketplace/apps/{appId}/analytics` | Get install analytics |
| GET | `/marketplace/apps/{appId}/analytics/usage` | Get usage metrics |
| GET | `/marketplace/apps/{appId}/analytics/ratings` | Get ratings data |

## Example: List Your Apps

```bash
curl "https://api.zoom.us/v2/marketplace/apps" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "apps": [
    {
      "id": "app_abc123",
      "name": "My Awesome App",
      "short_description": "Enhance your Zoom experience",
      "app_type": "zoom_app",
      "status": "published",
      "created_at": "2023-06-15T10:00:00Z",
      "published_at": "2023-07-01T10:00:00Z",
      "current_version": "2.1.0",
      "install_count": 5000,
      "average_rating": 4.5
    }
  ],
  "total_records": 1
}
```

## Example: Get App Details

```bash
curl "https://api.zoom.us/v2/marketplace/apps/{appId}" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "id": "app_abc123",
  "name": "My Awesome App",
  "short_description": "Enhance your Zoom experience",
  "long_description": "Full markdown description...",
  "app_type": "zoom_app",
  "status": "published",
  "visibility": "public",
  "categories": ["productivity", "collaboration"],
  "developer": {
    "name": "My Company",
    "email": "support@mycompany.com",
    "website": "https://mycompany.com",
    "privacy_policy": "https://mycompany.com/privacy",
    "terms_of_service": "https://mycompany.com/terms"
  },
  "scopes": [
    "meeting:read",
    "meeting:write",
    "user:read"
  ],
  "redirect_uris": [
    "https://myapp.com/auth/callback"
  ],
  "install_url": "https://marketplace.zoom.us/apps/app_abc123",
  "versions": {
    "current": "2.1.0",
    "draft": "2.2.0"
  },
  "analytics": {
    "total_installs": 5000,
    "active_installs": 4200,
    "average_rating": 4.5,
    "total_ratings": 150
  }
}
```

## Example: Create New Version

```bash
curl -X POST "https://api.zoom.us/v2/marketplace/apps/{appId}/versions" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "version": "2.2.0",
    "release_notes": "## What'\''s New\n- New dashboard feature\n- Bug fixes\n- Performance improvements",
    "changes": {
      "scopes_added": ["recording:read"],
      "scopes_removed": [],
      "features_added": ["Dashboard analytics"],
      "breaking_changes": false
    }
  }'
```

## Example: Submit for Review

```bash
curl -X POST "https://api.zoom.us/v2/marketplace/apps/{appId}/versions/{versionId}/submit" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "submission_notes": "Ready for review. All tests passing.",
    "test_credentials": {
      "email": "reviewer@zoom.com",
      "instructions": "Login with provided credentials to test all features."
    }
  }'
```

## App Status

| Status | Description |
|--------|-------------|
| `draft` | Not yet submitted |
| `submitted` | Awaiting review |
| `in_review` | Currently being reviewed |
| `changes_requested` | Reviewer requested changes |
| `approved` | Approved, ready to publish |
| `published` | Live on Marketplace |
| `rejected` | Review rejected |
| `deprecated` | Marked for removal |

## App Types

| Type | Description |
|------|-------------|
| `zoom_app` | Zoom Apps (in-meeting experience) |
| `team_chat_app` | Team Chat integration |
| `chatbot` | Team Chat chatbot |
| `webhook_only` | Webhook integration |
| `oauth` | OAuth-only integration |

## Visibility Options

| Visibility | Description |
|------------|-------------|
| `public` | Listed on Marketplace |
| `private` | Only accessible via direct link |
| `account` | Only for specific accounts |

## Analytics Data

### Get Install Analytics

```bash
curl "https://api.zoom.us/v2/marketplace/apps/{appId}/analytics?from=2024-01-01&to=2024-01-31" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "period": {
    "from": "2024-01-01",
    "to": "2024-01-31"
  },
  "summary": {
    "total_installs": 500,
    "total_uninstalls": 50,
    "net_installs": 450,
    "active_users": 4200
  },
  "daily_data": [
    {
      "date": "2024-01-01",
      "installs": 20,
      "uninstalls": 2,
      "active_users": 4000
    }
  ]
}
```

## Review Process

1. **Submit** - Submit version for review
2. **Automated Checks** - Security and compliance scans
3. **Manual Review** - Zoom team reviews functionality
4. **Feedback** - Receive feedback or approval
5. **Publish** - Publish approved version

### Review Timeline

| Submission Type | Typical Review Time |
|-----------------|---------------------|
| New app | 5-10 business days |
| Major update | 3-5 business days |
| Minor update | 1-3 business days |
| Bug fix | 1-2 business days |

## Update App Listing

```bash
curl -X PATCH "https://api.zoom.us/v2/marketplace/apps/{appId}" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "short_description": "Updated description",
    "categories": ["productivity", "automation"],
    "support_email": "newsupport@mycompany.com",
    "screenshots": [
      {
        "url": "https://mycompany.com/screenshot1.png",
        "caption": "Main dashboard"
      }
    ]
  }'
```

## Required Assets

| Asset | Requirements |
|-------|--------------|
| App icon | 512x512 PNG |
| Screenshots | 1280x800 or 800x1280 |
| Privacy policy | HTTPS URL |
| Terms of service | HTTPS URL |
| Support URL | HTTPS URL |

## Webhooks

| Event | Description |
|-------|-------------|
| `app.installed` | App installed by user/account |
| `app.uninstalled` | App uninstalled |
| `app.activated` | App activated |
| `app.deactivated` | App deactivated |

## Best Practices

1. **Test thoroughly** - Ensure all features work before submission
2. **Write clear descriptions** - Help users understand your app
3. **Provide test credentials** - Make reviewer's job easier
4. **Respond quickly** - Address feedback promptly
5. **Version semantically** - Use semantic versioning
6. **Document changes** - Write detailed release notes

## Resources

- **Marketplace Guide**: https://developers.zoom.us/docs/distribute/
- **Review Guidelines**: https://developers.zoom.us/docs/distribute/app-submission/review-guidelines/
- **Marketing Resources**: https://developers.zoom.us/docs/distribute/marketing/
- **Developer Dashboard**: https://marketplace.zoom.us/develop
