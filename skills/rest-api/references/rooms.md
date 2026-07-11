# Zoom Rooms API

Authoritative endpoint inventory for Rooms. This file mirrors the official Zoom API Hub OpenAPI document for this product area.

## Canonical Source

- OpenAPI JSON: https://developers.zoom.us/api-hub/rooms/methods/endpoints.json
- Base URL: `https://api.zoom.us/v2`
- Authentication details: [authentication.md](authentication.md)

## Notes

- Endpoint methods and paths below are generated from the official Zoom API Hub `paths` object.
- Scope names are defined per operation and frequently use granular scope names. Check the API Hub operation page for the exact scopes before implementation.
- Use this file for endpoint discovery and inventory. Use `../examples/` for orchestration patterns, not as the canonical source of path names.

## Coverage

| Metric | Value |
|--------|-------|
| Endpoint operations | 127 |
| Path templates | 75 |
| Tags | 10 |

## Tag Index

| Tag | Operations |
|-----|------------|
| Room Apps | 1 |
| Visitor Management | 6 |
| Workspaces | 39 |
| Zoom Rooms | 20 |
| Zoom Rooms Account | 4 |
| Zoom Rooms Calendar | 7 |
| Zoom Rooms Content | 31 |
| Zoom Rooms Devices | 2 |
| Zoom Rooms Location | 10 |
| Zoom Rooms Tags | 7 |

## Endpoints by Tag

### Room Apps

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/rooms/controller/apps/config` | Config Zoom Room Controller Apps | `ConfigZoomRoomControllerApps` |

### Visitor Management

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/visitor/invitation` | Get a list of visitors by location | `invitationList` |
| POST | `/visitor/invitation` | Send an invitation | `createInvitation` |
| GET | `/visitor/invitation/{invitationId}` | Invitation details by invitationID | `getInvitation` |
| DELETE | `/visitor/invitation/{invitationId}` | Delete an Invitation | `deleteInvitation` |
| PATCH | `/visitor/invitation/{invitationId}` | Update an invitation | `updateInvitation` |
| POST | `/visitor/invitation/{invitationId}/checkin` | Check in a visitor | `checkinVisitor` |

### Workspaces

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/workspaces` | List workspaces | `listWorkspaces` |
| POST | `/workspaces` | Create a workspace | `createWorkspace` |
| GET | `/workspaces/additional_informations` | List workspace additional information with time range | `Getaworkspaceadditionalenhancements` |
| GET | `/workspaces/assets` | Get all workspace assets | `Getallworkspaceassets` |
| POST | `/workspaces/assets` | Create a workspace asset | `AddaWorkspaceasset` |
| GET | `/workspaces/assets/{assetId}` | Get a workspace asset | `get_workspace_asset` |
| DELETE | `/workspaces/assets/{assetId}` | Delete a workspace asset | `DeleteaWorkspaceasset` |
| PATCH | `/workspaces/assets/{assetId}` | Edit a workspace asset | `PatchWorkspaceasset` |
| POST | `/workspaces/events` | Check in/out of a reservation | `reservationEvent` |
| POST | `/workspaces/floormap/files` | Add or Update a Workspace floor map | `AddOrUpdateAWorkspaceFloorMap` |
| GET | `/workspaces/floors/{floorId}/reservations` | Get a floor's reservations | `Getafloor'sreservations` |
| GET | `/workspaces/locations/{locationId}/floor_map` | Get a workspace location floor map | `Getlocationfloormap` |
| GET | `/workspaces/locations/{locationId}/neighborhoods` | List neighborhoods | `ListNeighborhoods` |
| DELETE | `/workspaces/locations/{locationId}/neighborhoods` | Delete neighborhoods | `DeleteNeighborhoods` |
| GET | `/workspaces/locations/{locationId}/neighborhoods/{neighborhoodId}` | Get a neighborhood | `GetNeighborhood` |
| GET | `/workspaces/questionnaires_summary` | Get questionnaire answer summary | `Getworkspacequestionnaireresponses` |
| GET | `/workspaces/released_workspaces_by_timeout` | List released workspaces by timeout | `getWorksapceReservationReleaseInof` |
| GET | `/workspaces/reservations` | List workspace reservations by locations | `Getworkspacereservationsinlocations` |
| PATCH | `/workspaces/settings` | Update workspace settings | `updateWorkspaceSettings` |
| POST | `/workspaces/settings/photos` | Add a workspace photo | `AddaWorkspacephoto` |
| DELETE | `/workspaces/settings/photos` | Delete workspace photos | `DeleteaWorkspacephoto` |
| GET | `/workspaces/usage` | Get a location's hot desk usage | `getHotDeskUsage` |
| GET | `/workspaces/users/{userId}/calendar/settings` | Get Workspace Calendar Free/Busy Event | `GetWorkspaceCalendarFree/BusyEvent` |
| POST | `/workspaces/users/{userId}/calendar/settings` | Set Workspace Calendar Free/Busy Event | `SetCalendarFree/BusyEvent` |
| GET | `/workspaces/users/{userId}/reservations` | Get a user's workspace's reservations | `userListReservations` |
| DELETE | `/workspaces/{locationId}/background` | Delete Workspace floor map | `DeleteWorkspaceFloorMap` |
| GET | `/workspaces/{workspaceId}` | Get a workspace | `getWorkspace` |
| DELETE | `/workspaces/{workspaceId}` | Delete a workspace | `deleteWorkspace` |
| PATCH | `/workspaces/{workspaceId}` | Update a workspace | `updateWorkspace` |
| GET | `/workspaces/{workspaceId}/assignment` | Get a desk assignment | `Getadeskassignment` |
| PUT | `/workspaces/{workspaceId}/assignment` | Set a desk assignment | `setADeskAssignment` |
| DELETE | `/workspaces/{workspaceId}/assignment` | Delete a desk assignment | `Deleteadeskassignment` |
| GET | `/workspaces/{workspaceId}/qr_code` | Get a workspace QR code | `getWorkspaceQRCode` |
| GET | `/workspaces/{workspaceId}/reservations` | Get a workspace's reservations | `listReservations` |
| POST | `/workspaces/{workspaceId}/reservations` | Create a reservation | `createReservation` |
| GET | `/workspaces/{workspaceId}/reservations/{reservationId}` | Get a workspace reservation by reservationId | `GETGetaworkspacereservationbyreservationID` |
| DELETE | `/workspaces/{workspaceId}/reservations/{reservationId}` | Delete a reservation | `deleteReservation` |
| PATCH | `/workspaces/{workspaceId}/reservations/{reservationId}` | Update a reservation | `updateReservation` |
| GET | `/workspaces/{workspaceId}/reservations/{reservationId}/questionnaires` | List workspace reservation questionnaires | `Listworkspacereservationquestionnaires` |

### Zoom Rooms

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/rooms` | List Zoom Rooms | `listZoomRooms` |
| POST | `/rooms` | Add a Zoom Room | `addARoom` |
| GET | `/rooms/digital_signage` | List digital signage contents | `listDigitalSignageContent` |
| PATCH | `/rooms/events` | Update E911 digital signage | `manageE911signage` |
| PATCH | `/rooms/{id}/events` | Use Zoom Room controls | `ZoomRoomsControls` |
| GET | `/rooms/{id}/settings` | Get Zoom Room settings | `getZRSettings` |
| PATCH | `/rooms/{id}/settings` | Update Zoom Room settings | `updateZRSettings` |
| GET | `/rooms/{roomId}` | Get Zoom Room profile | `getZRProfile` |
| DELETE | `/rooms/{roomId}` | Delete a Zoom Room | `deleteAZoomRoom` |
| PATCH | `/rooms/{roomId}` | Update a Zoom Room profile | `updateRoomProfile` |
| GET | `/rooms/{roomId}/device_profiles` | List device profiles | `getRoomProfiles` |
| POST | `/rooms/{roomId}/device_profiles` | Create a device profile | `createRoomDeviceProfile` |
| GET | `/rooms/{roomId}/device_profiles/devices` | Get device information | `getRoomDevices` |
| GET | `/rooms/{roomId}/device_profiles/{deviceProfileId}` | Get a device profile | `getRoomProfile` |
| DELETE | `/rooms/{roomId}/device_profiles/{deviceProfileId}` | Delete a device profile | `deleteRoomProfile` |
| PATCH | `/rooms/{roomId}/device_profiles/{deviceProfileId}` | Update a device profile | `updateDeviceProfile` |
| GET | `/rooms/{roomId}/devices` | List Zoom Room devices | `listZRDevices` |
| PUT | `/rooms/{roomId}/location` | Change a Zoom Room's location | `changeZRLocation` |
| GET | `/rooms/{roomId}/sensor_data` | Get Zoom Room sensor data | `getZRSensorData` |
| GET | `/rooms/{roomId}/virtual_controller` | Get Zoom Rooms virtual controller URL | `getWebzrcUrl` |

### Zoom Rooms Account

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/rooms/account_profile` | Get Zoom Room account profile | `getZRAccountProfile` |
| PATCH | `/rooms/account_profile` | Update Zoom Room account profile | `updateZRAccProfile` |
| GET | `/rooms/account_settings` | Get Zoom Room account settings | `getZRAccountSettings` |
| PATCH | `/rooms/account_settings` | Update Zoom Room account settings | `updateZoomRoomAccSettings` |

### Zoom Rooms Calendar

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/rooms/calendar/services` | List calendar services | `getCalendarServices` |
| DELETE | `/rooms/calendar/services/{serviceId}` | Delete a calendar service | `deleteACalendarService` |
| GET | `/rooms/calendar/services/{serviceId}/resources` | List calendar resources by calendar service | `getCalendarResourcesByServiceId` |
| POST | `/rooms/calendar/services/{serviceId}/resources` | Add a calendar resource to a calendar service | `addACalendarResourceToCalendarService` |
| GET | `/rooms/calendar/services/{serviceId}/resources/{resourceId}` | Get a calendar resource by ID | `getCalendarResourceById` |
| DELETE | `/rooms/calendar/services/{serviceId}/resources/{resourceId}` | Delete a calendar resource | `deleteACalendarResource` |
| PUT | `/rooms/calendar/services/{serviceId}/sync` | Start calendar service sync process | `syncACalendarService` |

### Zoom Rooms Content

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/rooms/content/background/contents` | List Zoom Rooms background image library contents | `ListZoomRoomsbackgroundimagelibrarycontents` |
| GET | `/rooms/content/background/contents/{contentId}` | Get Zoom Rooms background image library content | `GetZoomRoomsBackgroundImageLibraryContent` |
| DELETE | `/rooms/content/background/contents/{contentId}` | Delete Zoom Rooms Background Image Library Content | `DeleteZoomRoomsBackgroundImageLibraryContent` |
| GET | `/rooms/content/background/defaults` | List default Zoom Rooms background image library contents | `ListDefaultZoomRoomsBackgroundImageLibrarycontents` |
| GET | `/rooms/content/background/folders` | List Zoom Rooms Background Image Library Folders | `ListZoomRoomsBackgroundLibraryFolders` |
| POST | `/rooms/content/background/folders` | Add Zoom Rooms Background Image Library Folder | `AddZoomRoomsBackgroundLibraryFolder` |
| GET | `/rooms/content/background/folders/{folderId}` | Get Zoom Rooms Background Image Library Folder | `GetZoomRoomsBackgroundLibraryFolder` |
| DELETE | `/rooms/content/background/folders/{folderId}` | Delete Zoom Rooms Background Image Library Folder | `DeleteZoomRoomsBackgroundLibraryFolder` |
| PATCH | `/rooms/content/background/folders/{folderId}` | Update Zoom Rooms Background Image Library Folder | `UpdateaZoomRoomsBackgroundLibraryFolderName` |
| GET | `/rooms/content/digital_signage/contents` | List Digital Signage content items | `GETListdigitalsignagecontentitems` |
| POST | `/rooms/content/digital_signage/contents` | Add a digital signage URL | `AddadigitalsignageURL` |
| GET | `/rooms/content/digital_signage/contents/{contentId}` | Get Digital Signage content item | `Getdigitalsignagecontentitem` |
| DELETE | `/rooms/content/digital_signage/contents/{contentId}` | Delete a Digital Signage content item | `Deleteadigitalsignagecontentitem` |
| PATCH | `/rooms/content/digital_signage/contents/{contentId}` | Update a Digital Signage content item attributes | `Updateadigitalsignagecontentitemattributes` |
| POST | `/rooms/content/digital_signage/folders` | Add a digital signage content folder | `Addadigitalsignagecontentfolder` |
| GET | `/rooms/content/digital_signage/folders/{folderId}` | Get Digital Signage content folder | `Getdigitalsignagecontentfolderdetails` |
| DELETE | `/rooms/content/digital_signage/folders/{folderId}` | Delete a Digital Signage content folder | `Deleteadigitalsignagecontentfolder` |
| PATCH | `/rooms/content/digital_signage/folders/{folderId}` | Update a digital signage content folder | `Updateadigitalsignagecontentfolder` |
| GET | `/rooms/content/digital_signage/playlists` | List Digital Signage library playlists | `ListDigitalSignagelibraryplaylists` |
| POST | `/rooms/content/digital_signage/playlists` | Add a Digital Signage library playlist | `AddaDigitalSignagelibraryplaylist` |
| GET | `/rooms/content/digital_signage/playlists/{playlistId}` | Get Digital Signage library playlist | `GetDigitalSignagelibraryplaylist` |
| DELETE | `/rooms/content/digital_signage/playlists/{playlistId}` | Delete Digital Signage library playlist | `DeleteDigitalSignagelibraryplaylist` |
| PATCH | `/rooms/content/digital_signage/playlists/{playlistId}` | Update a Digital Signage library playlist | `UpdateaDigitalSignagelibraryplaylist` |
| GET | `/rooms/content/digital_signage/playlists/{playlistId}/contents` | Get Digital Signage library playlist content items | `GetDigitalSignagelibraryplaylistcontentitems` |
| PUT | `/rooms/content/digital_signage/playlists/{playlistId}/contents` | Update Digital Signage library playlist content items | `UpdateDigitalSignagelibraryplaylistcontentitems` |
| GET | `/rooms/content/digital_signage/playlists/{playlistId}/rooms` | List Digital Signage library playlist published rooms | `ListDigitalSignagelibraryplaylistpublishedrooms` |
| PUT | `/rooms/content/digital_signage/playlists/{playlistId}/rooms` | Update Digital Signage library playlist published rooms | `UpdateDigitalSignagelibraryplaylistpublishedrooms` |
| PUT | `/zrbackground/files` | Update Zoom Rooms background image library content | `UpdateZoomRoomsBackgroundImageLibraryContent` |
| POST | `/zrbackground/files` | Add Zoom Rooms background image library content | `AddZoomRoomsBackgroundImageLibraryContent` |
| PUT | `/zrdigitalsignage/files` | Update a Digital Signage image or video file | `Updateadigitalsignageimageorvideofile` |
| POST | `/zrdigitalsignage/files` | Add a digital signage image or video | `Adddigitalsignageimageorvideo` |

### Zoom Rooms Devices

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| DELETE | `/rooms/{roomId}/devices/{deviceId}` | Delete a Zoom Room user device | `deleteDevice` |
| PUT | `/rooms/{roomId}/devices/{deviceId}/app_version` | Change Zoom Rooms app version | `changeZoomRoomsAppVersion` |

### Zoom Rooms Location

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/rooms/locations` | List Zoom Room locations | `listZRLocations` |
| POST | `/rooms/locations` | Add a location | `addAZRLocation` |
| GET | `/rooms/locations/structure` | Get Zoom Room location structure | `getZRLocationStructure` |
| PUT | `/rooms/locations/structure` | Update Zoom Rooms location structure | `updateZoomRoomsLocationStructure` |
| GET | `/rooms/locations/{locationId}` | Get Zoom Room location profile | `getZRLocationProfile` |
| DELETE | `/rooms/locations/{locationId}` | Delete a location | `deleteAZRLocation` |
| PATCH | `/rooms/locations/{locationId}` | Update Zoom Room location profile | `updateZRLocationProfile` |
| PUT | `/rooms/locations/{locationId}/location` | Change the assigned parent location | `changeParentLocation` |
| GET | `/rooms/locations/{locationId}/settings` | Get location settings | `getZRLocationSettings` |
| PATCH | `/rooms/locations/{locationId}/settings` | Update location settings | `updateZRLocationSettings` |

### Zoom Rooms Tags

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| PATCH | `/rooms/locations/{locationId}/tags` | Assign Tags to Zoom Rooms By Location ID | `AssignTagsToZoomRoomsByLocationID` |
| GET | `/rooms/tags` | List all Zoom Room Tags | `listZoomRoomTags` |
| POST | `/rooms/tags` | Create a new Zoom Rooms Tag | `createZoomRoomTag` |
| DELETE | `/rooms/tags/{tagId}` | Delete Tag | `deleteZoomRoomTag` |
| PATCH | `/rooms/tags/{tagId}` | Edit Tag | `editZoomRoomTag` |
| DELETE | `/rooms/{roomId}/tags` | Un-assign Tags from a Zoom Room | `unassignZoomRoomTag` |
| PATCH | `/rooms/{roomId}/tags` | Assign Tags to a Zoom Room | `AssignTagsToAZoomRoom` |
