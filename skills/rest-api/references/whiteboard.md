# Zoom Whiteboard API

Authoritative endpoint inventory for Whiteboard. This file mirrors the official Zoom API Hub OpenAPI document for this product area.

## Canonical Source

- OpenAPI JSON: https://developers.zoom.us/api-hub/whiteboard/methods/endpoints.json
- Base URL: `https://api.zoom.us/v2`
- Authentication details: [authentication.md](authentication.md)

## Notes

- Endpoint methods and paths below are generated from the official Zoom API Hub `paths` object.
- Scope names are defined per operation and frequently use granular scope names. Check the API Hub operation page for the exact scopes before implementation.
- Use this file for endpoint discovery and inventory. Use `../examples/` for orchestration patterns, not as the canonical source of path names.

## Coverage

| Metric | Value |
|--------|-------|
| Endpoint operations | 43 |
| Path templates | 25 |
| Tags | 10 |

## Tag Index

| Tag | Operations |
|-----|------------|
| Archiving | 3 |
| Classification Labels | 7 |
| Collaborator | 4 |
| Content | 3 |
| Document | 5 |
| Export | 3 |
| File | 2 |
| Import | 2 |
| Project | 13 |
| Settings | 1 |

## Endpoints by Tag

### Archiving

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/whiteboards/sessions` | List whiteboards sessions | `Createwhiteboardsarchivefiles` |
| GET | `/whiteboards/sessions/activity/download/{path}` | Download Whiteboards activity file | `Downloadwhiteboardsactivityfile` |
| GET | `/whiteboards/sessions/{seesionId}` | List whiteboard sessions activities | `Listwhiteboardsessionsarchivedfiles` |

### Classification Labels

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/whiteboards/classification_labels` | List classification labels | `listClassificationLabels` |
| POST | `/whiteboards/classification_labels` | Create a classification label | `createClassificationLabel` |
| GET | `/whiteboards/classification_labels/{classificationId}` | Get classification label | `getClassificationLabel` |
| DELETE | `/whiteboards/classification_labels/{classificationId}` | Delete classification label | `deleteClassificationLabel` |
| PATCH | `/whiteboards/classification_labels/{classificationId}` | Update a classification label | `updateClassificationLabel` |
| PUT | `/whiteboards/{whiteboardId}/classification` | Apply classification to whiteboard | `Applyclassificationtowhiteboard` |
| DELETE | `/whiteboards/{whiteboardId}/classification` | Remove classification from whiteboard | `removeWhiteboardClassification` |

### Collaborator

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/whiteboards/{whiteboardId}/collaborator` | Get collaborators of a whiteboard | `GetAWhiteboardCollaborator` |
| POST | `/whiteboards/{whiteboardId}/collaborator` | Share a whiteboard to new users or team chat channels. | `AddAWhiteboardCollaborator` |
| PATCH | `/whiteboards/{whiteboardId}/collaborator` | Update whiteboard collaborators | `UpdateAWhiteboardCollaborator` |
| DELETE | `/whiteboards/{whiteboardId}/collaborator/{collaboratorId}` | Remove the collaborator from a whiteboard | `DeleteAWhiteboardCollaborator` |

### Content

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/whiteboards/{whiteboardId}/content` | Get Whiteboard Content | `GetWhiteboardContent` |
| POST | `/whiteboards/{whiteboardId}/content` | Save Whiteboard Content | `SaveWhiteboardContent` |
| DELETE | `/whiteboards/{whiteboardId}/content` | Delete Whiteboard Content | `DeleteWhiteboardContent` |

### Document

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/whiteboards` | List all whiteboards | `ListWhiteboards` |
| POST | `/whiteboards` | Create a new whiteboard | `newWhiteboardCreate` |
| GET | `/whiteboards/{whiteboardId}` | Get a whiteboard | `GetAWhiteboard` |
| PUT | `/whiteboards/{whiteboardId}` | Update whiteboard basic information | `UpdateAWhiteboardMetadata` |
| DELETE | `/whiteboards/{whiteboardId}` | Delete a whiteboard | `DeleteAWhiteboard` |

### Export

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/whiteboards/export` | Create whiteboard export | `Createwhiteboardsexport` |
| GET | `/whiteboards/export/task/{taskId}` | Download whiteboard export | `Downloadwhiteboardexport` |
| GET | `/whiteboards/export/task/{taskId}/status` | Get whiteboard export generation status | `Getwhiteboardexportdatagenerationstatus` |

### File

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/whiteboards/files` | Upload file for whiteboard import | `Uploadfileforwhiteboardimport` |
| GET | `/whiteboards/{whiteboardId}/files/{fileId}` | Download Imported Whiteboard File | `Downloadembeddedwhiteboardfile` |

### Import

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/whiteboards/import` | Create a new whiteboard by import | `CreateWhiteboardImport` |
| GET | `/whiteboards/import/{taskId}/status` | Get whiteboard import status | `GetWhiteboardimportstatus` |

### Project

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/whiteboards/projects` | List all projects | `Listallprojects` |
| POST | `/whiteboards/projects` | Create a new project | `Createproject` |
| GET | `/whiteboards/projects/{projectId}` | Get a project | `Getaproject` |
| DELETE | `/whiteboards/projects/{projectId}` | Delete a project | `Deleteproject` |
| PATCH | `/whiteboards/projects/{projectId}` | Update project basic information | `Updateproject` |
| GET | `/whiteboards/projects/{projectId}/collaborators` | Get collaborators of a project | `Getcollaboratorsofaproject` |
| POST | `/whiteboards/projects/{projectId}/collaborators` | Share a project to new users | `Shareaprojecttonewusers` |
| PATCH | `/whiteboards/projects/{projectId}/collaborators` | Update project collaborators | `Updateprojectcollaborators` |
| DELETE | `/whiteboards/projects/{projectId}/collaborators/{collaboratorId}` | Remove the collaborator from a project | `Removethecollaboratorfromaproject` |
| GET | `/whiteboards/projects/{projectId}/subprojects` | List subprojects | `listSubProjects` |
| POST | `/whiteboards/projects/{projectId}/subprojects` | Create a subproject | `createSubProject` |
| POST | `/whiteboards/projects/{projectId}/whiteboards` | Move whiteboards to a project | `Movewhiteboardstoproject` |
| DELETE | `/whiteboards/projects/{projectId}/whiteboards` | Remove whiteboards from a project | `Removewhiteboardsfromaproject` |

### Settings

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| PATCH | `/whiteboards/{whiteboardId}/share_setting` | Update whiteboard share setting | `UpdateAWhiteboardShareSetting` |
