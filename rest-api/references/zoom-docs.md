# Zoom Docs API

Document collaboration and management API.

## Overview

Zoom Docs provides collaborative document editing integrated with Zoom Workplace. The API enables creating, editing, and managing documents with real-time collaboration features.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with docs scopes:
- `docs:read` - Read documents
- `docs:write` - Create/edit documents
- `docs:read:admin` - Admin read access
- `docs:write:admin` - Admin write access

## Key Endpoints

### Documents

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/docs` | List documents |
| POST | `/docs` | Create document |
| GET | `/docs/{docId}` | Get document details |
| PATCH | `/docs/{docId}` | Update document metadata |
| DELETE | `/docs/{docId}` | Delete document |
| GET | `/docs/{docId}/content` | Get document content |
| PUT | `/docs/{docId}/content` | Update document content |

### Sharing & Permissions

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/docs/{docId}/sharing` | Get sharing settings |
| PATCH | `/docs/{docId}/sharing` | Update sharing settings |
| GET | `/docs/{docId}/collaborators` | List collaborators |
| POST | `/docs/{docId}/collaborators` | Add collaborators |
| DELETE | `/docs/{docId}/collaborators/{userId}` | Remove collaborator |

### Folders

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/docs/folders` | List folders |
| POST | `/docs/folders` | Create folder |
| PATCH | `/docs/folders/{folderId}` | Update folder |
| DELETE | `/docs/folders/{folderId}` | Delete folder |
| POST | `/docs/{docId}/move` | Move document to folder |

### Comments

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/docs/{docId}/comments` | List comments |
| POST | `/docs/{docId}/comments` | Add comment |
| PATCH | `/docs/{docId}/comments/{commentId}` | Update comment |
| DELETE | `/docs/{docId}/comments/{commentId}` | Delete comment |
| POST | `/docs/{docId}/comments/{commentId}/resolve` | Resolve comment |

### Version History

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/docs/{docId}/versions` | List versions |
| GET | `/docs/{docId}/versions/{versionId}` | Get specific version |
| POST | `/docs/{docId}/restore` | Restore to version |

### Export

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/docs/{docId}/export` | Export document |
| GET | `/docs/{docId}/export/{exportId}` | Get export status |

## Example: Create a Document

```bash
curl -X POST "https://api.zoom.us/v2/docs" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Project Proposal",
    "folder_id": "folder_abc",
    "template_id": "template_xyz"
  }'
```

### Response

```json
{
  "id": "doc_abc123",
  "title": "Project Proposal",
  "owner_id": "user_xyz",
  "created_at": "2024-01-15T10:00:00Z",
  "modified_at": "2024-01-15T10:00:00Z",
  "share_url": "https://zoom.us/docs/abc123"
}
```

## Example: Add Collaborators

```bash
curl -X POST "https://api.zoom.us/v2/docs/{docId}/collaborators" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "collaborators": [
      {
        "email": "editor@example.com",
        "permission": "editor"
      },
      {
        "email": "viewer@example.com",
        "permission": "viewer"
      }
    ]
  }'
```

## Example: Export Document

```bash
curl -X POST "https://api.zoom.us/v2/docs/{docId}/export" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "format": "pdf"
  }'
```

### Response

```json
{
  "export_id": "exp_xyz",
  "status": "processing",
  "download_url": null
}
```

## Permission Levels

| Level | Description |
|-------|-------------|
| `owner` | Full control |
| `editor` | Can view and edit |
| `commenter` | Can view and comment |
| `viewer` | Read-only access |

## Export Formats

| Format | Description |
|--------|-------------|
| `pdf` | PDF document |
| `docx` | Microsoft Word |
| `md` | Markdown |
| `html` | HTML |

## Document Templates

Templates provide pre-formatted starting points:
- Meeting notes
- Project plans
- Team wikis
- Status reports

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Docs
- **Zoom Docs Documentation**: https://developers.zoom.us/docs/zoom-docs/
