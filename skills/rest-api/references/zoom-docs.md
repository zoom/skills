# Zoom Docs API

Authoritative endpoint inventory for Canvas. This file mirrors the official Zoom API Hub OpenAPI document for this product area.

## Canonical Source

- OpenAPI JSON: https://developers.zoom.us/api-hub/canvas/methods/endpoints.json
- Base URL: `https://api.zoom.us/v2`
- Authentication details: [authentication.md](authentication.md)

## Notes

- Endpoint methods and paths below are generated from the official Zoom API Hub `paths` object.
- Scope names are defined per operation and frequently use granular scope names. Check the API Hub operation page for the exact scopes before implementation.
- Use this file for endpoint discovery and inventory. Use `../examples/` for orchestration patterns, not as the canonical source of path names.

## Coverage

| Metric | Value |
|--------|-------|
| Endpoint operations | 26 |
| Path templates | 20 |
| Tags | 8 |

## Tag Index

| Tag | Operations |
|-----|------------|
| Archiving | 2 |
| Collaborator | 4 |
| Doc Content | 4 |
| Export | 3 |
| File Management | 7 |
| File Uploads | 1 |
| General Access | 2 |
| Import | 3 |

## Endpoints by Tag

### Archiving

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/docs/archive_attachments` | Download archive attachments | `BatchGetArchiveAttachmentDownloadUrl` |
| GET | `/docs/archives` | List doc archive data | `Listdocarchive` |

### Collaborator

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/docs/files/{fileId}/collaborators` | List collaborators of a file | `ListCollaborators` |
| POST | `/docs/files/{fileId}/collaborators` | Add collaborators for a file | `AddCollaborators` |
| DELETE | `/docs/files/{fileId}/collaborators/{collaboratorId}` | Remove a collaborator from a file | `RemoveACollaborator` |
| PATCH | `/docs/files/{fileId}/collaborators/{collaboratorId}` | Modify a collaborator’s role on a file | `ModifyCollaboratorRole` |

### Doc Content

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/docs/files/{fileId}/blocks` | Insert blocks into a document | `Insertblocksintoadocument` |
| POST | `/docs/files/{fileId}/blocks/replace_range` | Replace a range of blocks | `Replacearangeofblocks` |
| DELETE | `/docs/files/{fileId}/blocks/{blockId}` | Delete a specific block | `DeleteBlock` |
| PATCH | `/docs/files/{fileId}/blocks/{blockId}` | Update a specific block | `UpdateBlock` |

### Export

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/docs/exports` | Create a file export | `Createafileexport` |
| GET | `/docs/exports/{exportId}/status` | Get file export status | `Getfileexportstatus` |
| GET | `/docs/files/{fileId}/content` | Get file content | `Getfilecontent` |

### File Management

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/docs/files` | Create a new file | `CreateDoc` |
| GET | `/docs/files/{fileId}` | Get metadata of a file | `QueryFileMetadata` |
| DELETE | `/docs/files/{fileId}` | Delete a file | `DeleteFile` |
| PATCH | `/docs/files/{fileId}` | Modify metadata of a file | `ModifyMetadata` |
| GET | `/docs/files/{fileId}/children` | List all children of a file | `ListAllChildren` |
| PUT | `/docs/files/{fileId}/owner` | Transfer ownership of a file | `TransferOwnershipOfAFile` |
| GET | `/docs/users/{userId}/root` | Get user My docs info | `GetUserMyDocsID` |

### File Uploads

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/docs/file_uploads` | Create file upload for docs import or attachments | `Uploadfilefordocsimportorattachments` |

### General Access

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/docs/files/{fileId}/general_access_setting` | Get the general access setting of a file | `GetFileGeneralAccess` |
| PATCH | `/docs/files/{fileId}/general_access_setting` | Modify the general access setting of a file | `ModifyFileGeneralAccess` |

### Import

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/docs/import_content` | Create a new file from content | `createNewFileWithMarkdown` |
| POST | `/docs/imports` | Create a new file by import | `Createanewfilebyimport` |
| GET | `/docs/imports/{importId}/status` | Get file import status | `Getdocsfileimportstatus` |
