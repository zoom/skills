# Zoom Whiteboard API

Collaborative whiteboard management API.

## Overview

Zoom Whiteboard enables real-time visual collaboration with infinite canvas, templates, and shapes. The API provides endpoints for creating, managing, and sharing whiteboards programmatically.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with whiteboard scopes:
- `whiteboard:read` - Read whiteboard data
- `whiteboard:write` - Create/modify whiteboards
- `whiteboard:read:admin` - Admin read access
- `whiteboard:write:admin` - Admin write access

## Key Endpoints

### Whiteboards

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/whiteboards` | List whiteboards |
| POST | `/whiteboards` | Create whiteboard |
| GET | `/whiteboards/{whiteboardId}` | Get whiteboard details |
| PATCH | `/whiteboards/{whiteboardId}` | Update whiteboard |
| DELETE | `/whiteboards/{whiteboardId}` | Delete whiteboard |
| POST | `/whiteboards/{whiteboardId}/duplicate` | Duplicate whiteboard |

### Sharing & Permissions

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/whiteboards/{whiteboardId}/sharing` | Get sharing settings |
| PATCH | `/whiteboards/{whiteboardId}/sharing` | Update sharing settings |
| GET | `/whiteboards/{whiteboardId}/collaborators` | List collaborators |
| POST | `/whiteboards/{whiteboardId}/collaborators` | Add collaborators |
| DELETE | `/whiteboards/{whiteboardId}/collaborators/{userId}` | Remove collaborator |

### Folders

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/whiteboards/folders` | List folders |
| POST | `/whiteboards/folders` | Create folder |
| PATCH | `/whiteboards/folders/{folderId}` | Update folder |
| DELETE | `/whiteboards/folders/{folderId}` | Delete folder |

### Export

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/whiteboards/{whiteboardId}/export` | Export whiteboard |
| GET | `/whiteboards/{whiteboardId}/export/{exportId}` | Get export status |
| GET | `/whiteboards/{whiteboardId}/thumbnail` | Get thumbnail |

### Templates

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/whiteboards/templates` | List templates |
| POST | `/whiteboards/templates` | Create template from whiteboard |
| POST | `/whiteboards/from-template` | Create whiteboard from template |

## Example: Create a Whiteboard

```bash
curl -X POST "https://api.zoom.us/v2/whiteboards" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sprint Planning Board",
    "folder_id": "folder_abc"
  }'
```

### Response

```json
{
  "id": "wb_xyz123",
  "name": "Sprint Planning Board",
  "owner_id": "user_abc",
  "created_at": "2024-01-15T10:00:00Z",
  "modified_at": "2024-01-15T10:00:00Z",
  "share_url": "https://zoom.us/wb/xyz123",
  "thumbnail_url": "https://zoom.us/wb/thumb/xyz123"
}
```

## Example: Add Collaborators

```bash
curl -X POST "https://api.zoom.us/v2/whiteboards/{whiteboardId}/collaborators" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "collaborators": [
      {
        "email": "user1@example.com",
        "permission": "editor"
      },
      {
        "email": "user2@example.com",
        "permission": "viewer"
      }
    ]
  }'
```

## Example: Export Whiteboard

```bash
curl -X POST "https://api.zoom.us/v2/whiteboards/{whiteboardId}/export" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "format": "pdf",
    "quality": "high"
  }'
```

### Response

```json
{
  "export_id": "exp_abc123",
  "status": "processing"
}
```

## Permission Levels

| Level | Description |
|-------|-------------|
| `owner` | Full control including delete |
| `editor` | Can view and edit content |
| `commenter` | Can view and add comments |
| `viewer` | Can only view |

## Export Formats

| Format | Description |
|--------|-------------|
| `pdf` | PDF document |
| `png` | PNG image |
| `svg` | SVG vector image |

## Sharing Options

| Option | Description |
|--------|-------------|
| `private` | Only owner and collaborators |
| `organization` | All organization members |
| `public` | Anyone with link |

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Whiteboards
- **Whiteboard Documentation**: https://developers.zoom.us/docs/zoom-whiteboard/
