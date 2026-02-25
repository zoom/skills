# Zoom Clips API

Video clips creation and management API.

## Overview

Zoom Clips allows users to create, share, and manage short video recordings. The API provides endpoints for creating clips, managing clip libraries, and controlling sharing settings.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with clips scopes:
- `clips:read` - Read clips data
- `clips:write` - Create/manage clips
- `clips:read:admin` - Admin read access
- `clips:write:admin` - Admin write access

## Key Endpoints

### Clips

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/clips` | List user clips |
| POST | `/clips` | Create/upload a clip |
| GET | `/clips/{clipId}` | Get clip details |
| PATCH | `/clips/{clipId}` | Update clip metadata |
| DELETE | `/clips/{clipId}` | Delete a clip |
| GET | `/clips/{clipId}/download` | Download clip file |

### Clip Sharing

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/clips/{clipId}/sharing` | Get sharing settings |
| PATCH | `/clips/{clipId}/sharing` | Update sharing settings |
| POST | `/clips/{clipId}/share` | Share clip with users |
| DELETE | `/clips/{clipId}/share/{userId}` | Remove user access |

### Folders

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/clips/folders` | List clip folders |
| POST | `/clips/folders` | Create folder |
| PATCH | `/clips/folders/{folderId}` | Update folder |
| DELETE | `/clips/folders/{folderId}` | Delete folder |
| POST | `/clips/{clipId}/move` | Move clip to folder |

### Transcripts

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/clips/{clipId}/transcript` | Get clip transcript |
| PATCH | `/clips/{clipId}/transcript` | Update transcript |

### Analytics

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/clips/{clipId}/analytics` | Get clip view analytics |
| GET | `/clips/analytics` | Get clips analytics summary |

## Example: List User Clips

```bash
curl -X GET "https://api.zoom.us/v2/clips?page_size=30" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "page_size": 30,
  "next_page_token": "abc123",
  "clips": [
    {
      "id": "clip_xyz",
      "title": "Product Demo",
      "description": "Quick walkthrough of new features",
      "duration": 180,
      "created_at": "2024-01-15T10:30:00Z",
      "thumbnail_url": "https://zoom.us/clips/thumb/xyz",
      "share_url": "https://zoom.us/clips/share/xyz",
      "status": "ready",
      "view_count": 42
    }
  ]
}
```

## Example: Create a Clip

```bash
curl -X POST "https://api.zoom.us/v2/clips" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Feature Overview",
    "description": "Overview of Q1 features",
    "folder_id": "folder_abc"
  }'
```

### Response

```json
{
  "id": "clip_new123",
  "upload_url": "https://zoom.us/clips/upload/presigned-url",
  "upload_token": "upload_token_xyz"
}
```

## Example: Update Sharing Settings

```bash
curl -X PATCH "https://api.zoom.us/v2/clips/{clipId}/sharing" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "access_level": "organization",
    "password_protected": true,
    "password": "secure123",
    "allow_download": false,
    "expiration_date": "2024-03-01T00:00:00Z"
  }'
```

## Clip Status

| Status | Description |
|--------|-------------|
| `uploading` | Clip is being uploaded |
| `processing` | Clip is being processed |
| `ready` | Clip is ready to view |
| `failed` | Processing failed |

## Access Levels

| Level | Description |
|-------|-------------|
| `private` | Only owner can view |
| `internal` | Organization members only |
| `public` | Anyone with link |
| `specific_users` | Specific users/groups |

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Clips
- **Zoom Clips Documentation**: https://developers.zoom.us/docs/zoom-clips/
