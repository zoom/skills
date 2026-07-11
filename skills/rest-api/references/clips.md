# Zoom Clips API

Authoritative endpoint inventory for Clips. This file mirrors the official Zoom API Hub OpenAPI document for this product area.

## Canonical Source

- OpenAPI JSON: https://developers.zoom.us/api-hub/clips/methods/endpoints.json
- Base URL: `https://api.zoom.us/`
- Authentication details: [authentication.md](authentication.md)

## Notes

- Endpoint methods and paths below are generated from the official Zoom API Hub `paths` object.
- Scope names are defined per operation and frequently use granular scope names. Check the API Hub operation page for the exact scopes before implementation.
- Use this file for endpoint discovery and inventory. Use `../examples/` for orchestration patterns, not as the canonical source of path names.

## Coverage

| Metric | Value |
|--------|-------|
| Endpoint operations | 20 |
| Path templates | 15 |
| Tags | 7 |

## Tag Index

| Tag | Operations |
|-----|------------|
| Clips | 2 |
| Collaborator | 2 |
| Comment | 2 |
| Download | 1 |
| Single | 7 |
| Transfer | 2 |
| Upload | 4 |

## Endpoints by Tag

### Clips

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/clips` | List all clips | `GetUserClips` |
| GET | `/clips/{clipId}/collaborators` | Get collaborators of a clip | `GetClipCollaborators` |

### Collaborator

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/clips/{clipId}/collaborators` | Share a clip with new users | `CreateCollaborator` |
| DELETE | `/clips/{clipId}/collaborators` | Remove the collaborator from a clip | `DeleteCollaborator` |

### Comment

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/clips/{clipId}/comments` | List clip comments | `Listclipcomments` |
| DELETE | `/clips/{clipId}/comments/{commentId}` | Delete a comment | `Deleteacomment` |

### Download

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/clips/{clipId}/download` | Download a clip | `downloadClip` |

### Single

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/clips/{clipId}` | Get a clip | `GetClipById` |
| DELETE | `/clips/{clipId}` | Delete a clip(soft delete) | `DeleteClip` |
| PATCH | `/clips/{clipId}` | Update a clip's basic info | `UpdateClipById` |
| GET | `/clips/{clipId}/chapters` | Get Clip Chapters | `GetClipChapters` |
| POST | `/clips/{clipId}/chapters` | Create Clip Chapters | `createClipChapters` |
| POST | `/clips/{clipId}/duplicate` | Duplicate Clips | `DuplicateClips` |
| PATCH | `/clips/{clipId}/share_settings` | Update clips share setting | `Updateclipssharesetting` |

### Transfer

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/clips/transfers` | Transfer clips owner | `Transferclipsowner` |
| GET | `/clips/transfers/{taskId}` | Transfer task status check | `Transfertaskstatuscheck` |

### Upload

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/clips/files` | Upload clip file | `UploadClipFile` |
| POST | `/clips/files/multipart` | Upload clip multipart files | `UploadIqMultipartClipFile` |
| POST | `/clips/files/multipart/upload_events` | Initiate and complete the multipart file upload for a clip | `InitiateAndCompleteAClipMultipartUpload.` |
| POST | `/clips/files/tmp` | Temporary file upload API for Clips | `TemporaryfileuploadAPIforClips` |
