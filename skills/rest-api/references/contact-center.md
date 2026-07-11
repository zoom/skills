# Zoom Contact Center API

Authoritative endpoint inventory for Contact Center. This file mirrors the official Zoom API Hub OpenAPI document for this product area.

## Canonical Source

- OpenAPI JSON: https://developers.zoom.us/api-hub/contact-center/methods/endpoints.json
- Base URL: `https://api.zoom.us/v2`
- Authentication details: [authentication.md](authentication.md)

## Notes

- Endpoint methods and paths below are generated from the official Zoom API Hub `paths` object.
- Scope names are defined per operation and frequently use granular scope names. Check the API Hub operation page for the exact scopes before implementation.
- Use this file for endpoint discovery and inventory. Use `../examples/` for orchestration patterns, not as the canonical source of path names.

## Coverage

| Metric | Value |
|--------|-------|
| Endpoint operations | 314 |
| Path templates | 178 |
| Tags | 29 |

## Tag Index

| Tag | Operations |
|-----|------------|
| Account | 1 |
| Address Books | 24 |
| Agent Statuses | 5 |
| Asset Library | 13 |
| Block List Rules | 8 |
| Call Control | 3 |
| Campaigns | 28 |
| Dispositions | 10 |
| Engagements | 12 |
| Flows | 11 |
| Follow-up Tasks | 3 |
| Inboxes | 18 |
| Logs | 5 |
| Messaging | 1 |
| Notes | 4 |
| Operating Hours | 14 |
| Queues | 40 |
| Recordings | 4 |
| Regions | 7 |
| Reports V1 (Legacy) | 7 |
| Reports V2 (CX Analytics) | 14 |
| Roles | 10 |
| Routing Profiles | 10 |
| RTMS | 1 |
| Skills | 11 |
| SMS | 1 |
| Teams | 14 |
| Users | 22 |
| Variables | 13 |

## Endpoints by Tag

### Account

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/storage/locations` | List storage locations | `ListStorageLocations` |

### Address Books

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/address_books` | List address books | `listAddressBooks` |
| POST | `/contact_center/address_books` | Create an address book | `createAddressBook` |
| GET | `/contact_center/address_books/contacts/{contactId}/custom_fields` | List an address book contact's custom fields | `ListContactCustomFields` |
| GET | `/contact_center/address_books/custom_fields` | List an address book's custom fields | `Listaddressbookcustomfields` |
| POST | `/contact_center/address_books/custom_fields` | Create an address book custom field | `Createacustomfield` |
| GET | `/contact_center/address_books/custom_fields/{customFieldId}` | Get an address book's custom field | `Getaaddressbookcustomfield` |
| DELETE | `/contact_center/address_books/custom_fields/{customFieldId}` | Delete an address book custom field | `Deleteancustomfield` |
| PATCH | `/contact_center/address_books/custom_fields/{customFieldId}` | Update an address book custom field | `Updateacustomfield` |
| GET | `/contact_center/address_books/units` | List address book units | `listUnits` |
| POST | `/contact_center/address_books/units` | Create an address book unit | `createUnit` |
| GET | `/contact_center/address_books/units/{unitId}` | Get an address book unit | `getUnit` |
| DELETE | `/contact_center/address_books/units/{unitId}` | Delete an address book unit | `deleteUnit` |
| PATCH | `/contact_center/address_books/units/{unitId}` | Update an address book unit | `updateUnit` |
| GET | `/contact_center/address_books/{addressBookId}` | Get an address book | `getAddressBook` |
| DELETE | `/contact_center/address_books/{addressBookId}` | Delete an address book | `deleteAddressBook` |
| PATCH | `/contact_center/address_books/{addressBookId}` | Update an address book | `updateAddressBook` |
| GET | `/contact_center/address_books/{addressBookId}/contacts` | List address book contacts | `listContacts` |
| POST | `/contact_center/address_books/{addressBookId}/contacts` | Create an address book contact | `createContact` |
| POST | `/contact_center/address_books/{addressBookId}/contacts/batch` | Batch create address book contacts | `batchCreateContact` |
| DELETE | `/contact_center/address_books/{addressBookId}/contacts/batch` | Batch delete address book contacts | `batchDeletecontact` |
| PATCH | `/contact_center/address_books/{addressBookId}/contacts/batch` | Batch update address book contacts | `batchUpdateContact` |
| GET | `/contact_center/address_books/{addressBookId}/contacts/{contactId}` | Get an address book contact | `getContact` |
| DELETE | `/contact_center/address_books/{addressBookId}/contacts/{contactId}` | Delete an address book contact | `contactDelete` |
| PATCH | `/contact_center/address_books/{addressBookId}/contacts/{contactId}` | Update an address book contact | `updateContact` |

### Agent Statuses

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/system_statuses` | List system statuses | `listSystemStatus` |
| POST | `/contact_center/system_statuses` | Create a system status | `createSystemStatus` |
| GET | `/contact_center/system_statuses/{statusId}` | Get a system status | `getAStatus` |
| DELETE | `/contact_center/system_statuses/{statusId}` | Delete a system status | `deleteSystemStatus` |
| PATCH | `/contact_center/system_statuses/{statusId}` | Update a system status | `updateSystemStatus` |

### Asset Library

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/asset_library/assets` | List assets | `listAssets` |
| POST | `/contact_center/asset_library/assets` | Create an asset | `createAnAsset` |
| PATCH | `/contact_center/asset_library/assets/items` | Batch update asset items | `BatchUpdateAssetItems` |
| GET | `/contact_center/asset_library/assets/{assetId}` | Get an asset | `getAnAsset` |
| DELETE | `/contact_center/asset_library/assets/{assetId}` | Delete an asset | `deleteAnAsset` |
| PATCH | `/contact_center/asset_library/assets/{assetId}` | Update an asset | `updateAnAsset` |
| POST | `/contact_center/asset_library/assets/{assetId}/duplicate` | Duplicate an asset | `duplicateAnAsset` |
| DELETE | `/contact_center/asset_library/assets/{assetId}/items` | Delete asset items | `Deleteassetitems` |
| GET | `/contact_center/asset_library/categories` | List asset categories | `listAssetCategories` |
| POST | `/contact_center/asset_library/categories` | Create an asset category | `createAnAssetCategory` |
| GET | `/contact_center/asset_library/categories/{categoryId}` | Get an asset category | `getAnAssetCategory` |
| DELETE | `/contact_center/asset_library/categories/{categoryId}` | Delete an asset category | `deleteAnAssetCategory` |
| PATCH | `/contact_center/asset_library/categories/{categoryId}` | Update an asset category | `updateAnAssetCategory` |

### Block List Rules

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/block_list_rules/ip_addresses` | List block list rules for IP addresses | `listBlockListRulesIpAddresses` |
| POST | `/contact_center/block_list_rules/ip_addresses` | Create block list rules for IP addresses | `createBlockListRulesIpAddresses` |
| DELETE | `/contact_center/block_list_rules/ip_addresses` | Delete IP addresses' block list rules | `deleteBlockListRuleIpAddresses` |
| GET | `/contact_center/block_list_rules/ip_addresses/{ipAddressBlockListRuleId}` | Get a block list rule of IP addresses | `getBlockListRuleIpAddress` |
| GET | `/contact_center/block_list_rules/phone_numbers` | List block list rules of phone numbers | `listBlockListRulesPhoneNumbers` |
| POST | `/contact_center/block_list_rules/phone_numbers` | Create block list rules for phone numbers | `createBlockListRulesPhoneNumbers` |
| DELETE | `/contact_center/block_list_rules/phone_numbers` | Delete phone numbers' block list rules | `deleteBlockListRulePhoneNumbers` |
| GET | `/contact_center/block_list_rules/phone_numbers/{phoneBlockListRuleId}` | Get a block list rule of phone numbers | `getBlockListRulePhoneNumber` |

### Call Control

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| PUT | `/contact_center/engagements/{engagementId}/recording/{command}` | Control an engagement's recording | `engagementRecordingControl` |
| POST | `/contact_center/users/{userId}/commands` | Command control of a user | `userControl` |
| GET | `/contact_center/users/{userId}/devices` | List user's devices | `Listuserdevices` |

### Campaigns

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/outbound_campaign/campaigns` | List outbound campaigns | `listOutboundCampaigns` |
| POST | `/contact_center/outbound_campaign/campaigns` | Create an outbound campaign | `createOutboundCampaign` |
| GET | `/contact_center/outbound_campaign/campaigns/{campaignId}` | Get an outbound campaign | `getOutboundCampaign` |
| DELETE | `/contact_center/outbound_campaign/campaigns/{campaignId}` | Delete an outbound campaign | `deleteOutboundCampaign` |
| PATCH | `/contact_center/outbound_campaign/campaigns/{campaignId}` | Update an outbound campaign | `updateOutboundCampaign` |
| PATCH | `/contact_center/outbound_campaign/campaigns/{campaignId}/status` | Update an outbound campaign status | `Updateanoutboundcampaignstatus` |
| GET | `/contact_center/outbound_campaign/contact_list_custom_fields` | List campaign contact lists' custom fields | `listCampaignCustomFields` |
| POST | `/contact_center/outbound_campaign/contact_list_custom_fields` | Create a campaign contact list's custom field | `createACampaignContactCustomField` |
| GET | `/contact_center/outbound_campaign/contact_list_custom_fields/{customFieldId}` | Get a campaign contact list's custom field | `getACampaignContactCustomField` |
| DELETE | `/contact_center/outbound_campaign/contact_list_custom_fields/{customFieldId}` | Delete a campaign contact list's custom field | `deleteACampaignCustomField` |
| PATCH | `/contact_center/outbound_campaign/contact_list_custom_fields/{customFieldId}` | Update a campaign contact list's custom field | `updateACampaignCustomField` |
| GET | `/contact_center/outbound_campaign/contact_lists` | List campaign contact lists | `listCampaignContactLists` |
| POST | `/contact_center/outbound_campaign/contact_lists` | Create a campaign contact list | `createCampaignContactList` |
| PATCH | `/contact_center/outbound_campaign/contact_lists` | Batch update campaign contact lists | `Batchupdatecampaigncontactlists` |
| GET | `/contact_center/outbound_campaign/contact_lists/{contactListId}` | Get a campaign contact list | `getCampaignContactList` |
| DELETE | `/contact_center/outbound_campaign/contact_lists/{contactListId}` | Remove a campaign contact list | `deleteCampaignContactList` |
| PATCH | `/contact_center/outbound_campaign/contact_lists/{contactListId}` | Update a campaign contact list | `updateCampaignContactList` |
| GET | `/contact_center/outbound_campaign/contact_lists/{contactListId}/contacts` | List a campaign contact list's contacts | `listCampaignContactListContacts` |
| POST | `/contact_center/outbound_campaign/contact_lists/{contactListId}/contacts` | Create a campaign contact list's contact | `createCampaignContactListContact` |
| PATCH | `/contact_center/outbound_campaign/contact_lists/{contactListId}/contacts` | Update a batch of contacts on a campaign contact list | `Updateabatchofcontactsonacampaigncontactlist` |
| POST | `/contact_center/outbound_campaign/contact_lists/{contactListId}/contacts/batch` | Insert a batch of contacts into a campaign contact list | `Insertabatchofcontactstoacampaigncontactlist` |
| GET | `/contact_center/outbound_campaign/contact_lists/{contactListId}/contacts/{contactId}` | Get a campaign contact list's contact | `getCampaignContactListContact` |
| DELETE | `/contact_center/outbound_campaign/contact_lists/{contactListId}/contacts/{contactId}` | Remove campaign contact list's contact | `deleteCampaigncontactListContact` |
| PATCH | `/contact_center/outbound_campaign/contact_lists/{contactListId}/contacts/{contactId}` | Update contact on a campaign contact list | `updateCampaignContactListContact` |
| GET | `/contact_center/outbound_campaign/contacts/{contactId}/custom_fields` | List a campaign contact's custom fields | `listCampaignContactCustomFields` |
| GET | `/contact_center/outbound_campaign/dnc_lists/{dncListId}/phones` | List campaign DNC phone numbers | `listCampaignDncListPhones` |
| POST | `/contact_center/outbound_campaign/dnc_lists/{dncListId}/phones` | Batch create a campaign DNC list's phones | `batchCreateCampaignDncListPhones` |
| DELETE | `/contact_center/outbound_campaign/dnc_lists/{dncListId}/phones` | Batch delete a campaign DNC list's phones | `batchDeleteCampaignDncListPhone` |

### Dispositions

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/dispositions` | List dispositions | `listDispositions` |
| POST | `/contact_center/dispositions` | Create a disposition | `createDisposition` |
| GET | `/contact_center/dispositions/sets` | List disposition sets | `listSets` |
| POST | `/contact_center/dispositions/sets` | Create a disposition set | `createSet` |
| GET | `/contact_center/dispositions/sets/{dispositionSetId}` | Get a disposition set | `getSet` |
| DELETE | `/contact_center/dispositions/sets/{dispositionSetId}` | Delete a disposition set | `deleteSet` |
| PATCH | `/contact_center/dispositions/sets/{dispositionSetId}` | Update a disposition set | `updateSet` |
| GET | `/contact_center/dispositions/{dispositionId}` | Get a disposition | `getDisposition` |
| DELETE | `/contact_center/dispositions/{dispositionId}` | Delete a disposition | `deleteDisposition` |
| PATCH | `/contact_center/dispositions/{dispositionId}` | Update a disposition | `updateDisposition` |

### Engagements

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/contact_center/engagement` | Start an engagement | `Startworkitemengagement` |
| GET | `/contact_center/engagements` | List engagements | `listEngagements` |
| GET | `/contact_center/engagements/{engagementId}` | Get an engagement | `getEngagement` |
| PATCH | `/contact_center/engagements/{engagementId}` | Update an engagement | `updateEngagement` |
| GET | `/contact_center/engagements/{engagementId}/attachments` | Get an engagement's attachments | `ListAttachments` |
| GET | `/contact_center/engagements/{engagementId}/events` | Get an engagement's events | `getEngagementEvents` |
| GET | `/contact_center/engagements/{engagementId}/recordings/status` | Poll an engagement recording's status | `EngagementRecordingStatus` |
| GET | `/contact_center/engagements/{engagementId}/survey` | Get an engagement's survey | `getEngagementSurvey` |
| GET | `/contact_center/engagements/{engagementId}/transfer/flows` | Get available transfer flows | `GetAvailableTransferFlows` |
| GET | `/contact_center/engagements/{engagementId}/transfer/queues` | Get available transfer queues | `GetAvailableTransferQueues` |
| GET | `/contact_center/engagements/{engagementId}/transfer/users` | Get available transfer users | `GetAvailableTransferUsers` |
| GET | `/contact_center/open_engagements` | List open engagements | `list open engagements` |

### Flows

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/flows` | List flows | `listFlows` |
| POST | `/contact_center/flows` | Import a flow | `ImportFlow` |
| GET | `/contact_center/flows/{flowId}` | Get a flow | `getAFlow` |
| DELETE | `/contact_center/flows/{flowId}` | Delete a flow | `DeleteFlow` |
| PATCH | `/contact_center/flows/{flowId}` | Edit a flow | `EditFlow` |
| GET | `/contact_center/flows/{flowId}/entry_points` | List flow's entry points | `ListFlowEntryPoints` |
| POST | `/contact_center/flows/{flowId}/entry_points` | Add flow entry points | `AddFlowEntryPoints` |
| DELETE | `/contact_center/flows/{flowId}/entry_points` | Remove flow entry points | `RemoveFlowEntryPoints` |
| GET | `/contact_center/flows/{flowId}/export` | Export a flow | `ExportFlow` |
| PUT | `/contact_center/flows/{flowId}/publish` | Publish a flow | `PublishFlow` |
| GET | `/contact_center/flows_entry_points` | List entry points | `ListentryPoints` |

### Follow-up Tasks

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/engagements/{engagementId}/follow_up_tasks` | List engagement follow-up tasks | `engagementFollowUpTasksBatchList` |
| GET | `/contact_center/engagements/{engagementId}/follow_up_tasks/{taskId}` | Get a follow-up task | `engagementFollowUpTaskGet` |
| PATCH | `/contact_center/engagements/{engagementId}/follow_up_tasks/{taskId}` | Update a follow-up task | `engagementFollowUpTaskPatch` |

### Inboxes

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/inboxes` | List inboxes | `listInbox` |
| POST | `/contact_center/inboxes` | Create an inbox | `inboxCreate` |
| DELETE | `/contact_center/inboxes` | Delete inboxes | `inboxesDelete` |
| GET | `/contact_center/inboxes/messages` | List an account's inbox messages | `listInboxesMessages` |
| DELETE | `/contact_center/inboxes/messages` | Delete inbox messages | `inboxesMessagesDelete` |
| GET | `/contact_center/inboxes/{inboxId}` | Get an inbox | `getInbox` |
| PATCH | `/contact_center/inboxes/{inboxId}` | Update an inbox | `inboxUpdate` |
| PATCH | `/contact_center/inboxes/{inboxId}/email_notification` | Update an inbox's email notification setting | `Updateaninboxemailnotification` |
| GET | `/contact_center/inboxes/{inboxId}/email_notifications` | Get an inbox email notification list | `Getinboxemailnotificationlist` |
| GET | `/contact_center/inboxes/{inboxId}/messages` | List an inbox's messages | `listInboxMessages` |
| DELETE | `/contact_center/inboxes/{inboxId}/messages` | Delete an inbox's messages | `inboxMessagesDelete` |
| DELETE | `/contact_center/inboxes/{inboxId}/messages/{messageId}` | Delete an inbox message | `inboxMessageDelete` |
| GET | `/contact_center/inboxes/{inboxId}/queues` | Get inbox access queues | `listInboxQueues` |
| POST | `/contact_center/inboxes/{inboxId}/queues` | Assign inbox access queues | `assignInboxQueues` |
| DELETE | `/contact_center/inboxes/{inboxId}/queues` | Remove inbox access queues | `unassignInboxQueues` |
| GET | `/contact_center/inboxes/{inboxId}/users` | List inbox users | `listInboxUsers` |
| POST | `/contact_center/inboxes/{inboxId}/users` | Assign inbox access users | `assignInboxUsers` |
| DELETE | `/contact_center/inboxes/{inboxId}/users` | Unassign inbox access users | `unassignInboxUsers` |

### Logs

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/email/messages` | List email message history | `getEmailMessageHistory` |
| GET | `/contact_center/messaging/messages` | List message history | `getMessageHistory` |
| GET | `/contact_center/sms` | List SMS logs | `listSMS` |
| GET | `/contact_center/voice_calls` | List voice call logs | `listVoiceCall` |
| GET | `/contact_center/work_item/messages` | List work item message history | `getWorkItemMessageHistory` |

### Messaging

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/contact_center/messages` | Send a message | `SendaMessage` |

### Notes

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/engagements/notes` | List notes | `notes` |
| GET | `/contact_center/engagements/{engagementId}/notes` | List engagement notes | `engagementNotes` |
| GET | `/contact_center/engagements/{engagementId}/notes/{noteId}` | Get a note | `getNote` |
| PATCH | `/contact_center/engagements/{engagementId}/notes/{noteId}` | Update a note | `noteUpdate` |

### Operating Hours

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/business_hours` | List business hours | `listBusinessHours` |
| POST | `/contact_center/business_hours` | Create business hours | `businessHourCreate` |
| GET | `/contact_center/business_hours/{businessHourId}` | Get business hours | `getABusinessHour` |
| DELETE | `/contact_center/business_hours/{businessHourId}` | Delete business hours | `businessHourDelete` |
| PATCH | `/contact_center/business_hours/{businessHourId}` | Update business hours | `businessHourUpdate` |
| GET | `/contact_center/business_hours/{businessHourId}/flows` | List the business hours' flows | `listBusinessHourFlows` |
| GET | `/contact_center/business_hours/{businessHourId}/queues` | List the business hours' queues | `listBusinessHourQueues` |
| GET | `/contact_center/closures` | List closures | `listClosures` |
| POST | `/contact_center/closures` | Create a closure set | `closuresSetCreate` |
| GET | `/contact_center/closures/{closureSetId}` | Get a closure set | `getAClosureSet` |
| DELETE | `/contact_center/closures/{closureSetId}` | Delete a closure set | `closureSetDelete` |
| PATCH | `/contact_center/closures/{closureSetId}` | Update a closure set | `closureSetUpdate` |
| GET | `/contact_center/closures/{closureSetId}/flows` | List the closures' flows | `listClosureSetFlows` |
| GET | `/contact_center/closures/{closureSetId}/queues` | List the closures' queues | `listClosureSetQueues` |

### Queues

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/queue_templates` | List queue templates | `Listqueuetemplates` |
| POST | `/contact_center/queue_templates` | Create a queue template | `createQueueTemplate` |
| DELETE | `/contact_center/queue_templates` | Batch delete queue templates | `deleteQueueTemplates` |
| GET | `/contact_center/queue_templates/{queueTemplateId}` | Get a queue template | `getQueueTemplate` |
| DELETE | `/contact_center/queue_templates/{queueTemplateId}` | Delete a queue template | `deleteQueueTemplate` |
| PATCH | `/contact_center/queue_templates/{queueTemplateId}` | Update a queue template | `updateQueueTemplate` |
| GET | `/contact_center/queues` | List queues | `listQueues` |
| POST | `/contact_center/queues` | Create a queue | `queueCreate` |
| POST | `/contact_center/queues/batch` | Batch create queues with a template | `Batchcreatequeueswithatemplate` |
| DELETE | `/contact_center/queues/batch` | Batch delete queues | `Batchdeletequeues` |
| GET | `/contact_center/queues/{queueId}` | Get a queue | `getAQueue` |
| DELETE | `/contact_center/queues/{queueId}` | Delete a queue | `queueDelete` |
| PATCH | `/contact_center/queues/{queueId}` | Update a queue | `queueUpdate` |
| GET | `/contact_center/queues/{queueId}/agents` | List queue agents | `getQueueAgents` |
| POST | `/contact_center/queues/{queueId}/agents` | Assign queue agents | `assignQueueAgents` |
| DELETE | `/contact_center/queues/{queueId}/agents/{userId}` | Unassign a queue agent | `deleteQueueAgent` |
| PATCH | `/contact_center/queues/{queueId}/agents/{userId}` | Update a queue agent's opt-in/opt-out status | `updateQueueAgent` |
| GET | `/contact_center/queues/{queueId}/dispositions` | List queue dispositions | `getQueueDispositions` |
| POST | `/contact_center/queues/{queueId}/dispositions` | Assign queue dispositions | `assignQueueDispositions` |
| GET | `/contact_center/queues/{queueId}/dispositions/sets` | List queue disposition sets | `getQueueDispositionSets` |
| POST | `/contact_center/queues/{queueId}/dispositions/sets` | Assign queue disposition sets | `assignQueueDispositionSets` |
| DELETE | `/contact_center/queues/{queueId}/dispositions/sets/{dispositionSetId}` | Unassign a queue disposition set | `deleteQueueDispositionSet` |
| DELETE | `/contact_center/queues/{queueId}/dispositions/{dispositionId}` | Unassign a queue disposition | `deleteQueueDisposition` |
| GET | `/contact_center/queues/{queueId}/interrupt` | Get a queue's interrupt settings | `getQueueInterruptSettings` |
| PATCH | `/contact_center/queues/{queueId}/interrupt` | Update a queue's interrupt settings | `updateQueueInterrupts` |
| POST | `/contact_center/queues/{queueId}/interrupt_menu` | Assign a queue menu based interrupt | `assignQueueMenuBasedInterrupt` |
| DELETE | `/contact_center/queues/{queueId}/interrupt_menu` | Delete a queue's interrupt menu configuration | `deleteQueueInterruptMenu` |
| GET | `/contact_center/queues/{queueId}/operating_hours` | Get a queue's operating hours | `getAQueueOperatingHours` |
| PATCH | `/contact_center/queues/{queueId}/operating_hours` | Update a queue's operating hours | `QueueOperatingHoursUpdate` |
| GET | `/contact_center/queues/{queueId}/recordings` | List queue recordings | `listQueueRecordings` |
| DELETE | `/contact_center/queues/{queueId}/recordings` | Delete queue recordings | `deleteQueueRecordings` |
| DELETE | `/contact_center/queues/{queueId}/scheduled_callbacks/attendees/{attendeeId}` | Delete an attendee from a scheduled callback event | `Deleteascheduledcallbackforanattendee` |
| POST | `/contact_center/queues/{queueId}/scheduled_callbacks/events` | Schedule a callback on a queue | `Scheduleacallbackonaqueue` |
| GET | `/contact_center/queues/{queueId}/scheduled_callbacks/supportive_slots` | List a queue's scheduled callbacks availability | `Listqueuescheduledcallbacksavailability` |
| GET | `/contact_center/queues/{queueId}/supervisors` | List queue supervisors | `getQueueSupervisors` |
| POST | `/contact_center/queues/{queueId}/supervisors` | Assign queue supervisors | `assignQueueSupervisors` |
| DELETE | `/contact_center/queues/{queueId}/supervisors/{userId}` | Unassign a queue supervisor | `deleteQueueSupervisor` |
| POST | `/contact_center/queues/{queueId}/teams` | Assign queue teams | `assignQueueTeams` |
| DELETE | `/contact_center/queues/{queueId}/teams` | Unassign queue teams | `batchDeleteQueueTeams` |
| DELETE | `/contact_center/queues/{queueId}/teams/{teamId}` | Unassign a queue team | `deleteQueueTeam` |

### Recordings

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/engagements/{engagementId}/recordings` | List engagement recordings | `listEngagementRecordings` |
| DELETE | `/contact_center/engagements/{engagementId}/recordings` | Delete engagement recordings | `deleteEngagementRecordings` |
| GET | `/contact_center/recordings` | List recordings | `listRecordings` |
| DELETE | `/contact_center/recordings/{recordingId}` | Delete a recording | `deleteRecording` |

### Regions

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/regions` | List regions | `ListRegions` |
| POST | `/contact_center/regions` | Create a region | `CreateARegion` |
| GET | `/contact_center/regions/{regionId}` | Get a region | `GetARegion` |
| DELETE | `/contact_center/regions/{regionId}` | Delete a region | `DeleteARegion` |
| PATCH | `/contact_center/regions/{regionId}` | Update a region | `UpdateARegion` |
| GET | `/contact_center/regions/{regionId}/users` | List a region's users | `ListRegion'sUsers` |
| POST | `/contact_center/regions/{regionId}/users` | Assign users to a region | `AssignUsersToARegion` |

### Reports V1 (Legacy)

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/analytics/agents/leg_metrics` | List agent leg reports | `listAgentLegMetric` |
| GET | `/contact_center/analytics/agents/status_history` | List agent's status history reports | `listAgentStatusHistory` |
| GET | `/contact_center/analytics/agents/time_sheets` | List agent's time sheet reports | `listAgentTimeSheet` |
| GET | `/contact_center/analytics/historical/details/metrics` | List historical detail reports | `listHistoricalDetailMetric` |
| GET | `/contact_center/analytics/historical/queues/agents/metrics` | List historical agent reports by queue | `listQueueAgentsMetrics` |
| GET | `/contact_center/analytics/historical/queues/metrics` | List historical queue reports | `listHistoricalQueueMetric` |
| GET | `/contact_center/analytics/historical/queues/{queueId}/agents/metrics` | List historical queue's agents reports | `listQueueAgentMetric` |

### Reports V2 (CX Analytics)

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/analytics/dataset/historical/agent_performance` | List historical agent performance dataset data | `Listhistoricalagentperformancedatasetdata` |
| GET | `/contact_center/analytics/dataset/historical/agent_timecard` | List historical agent timecard dataset data | `Listhistoricalagenttimecarddatasetdata` |
| GET | `/contact_center/analytics/dataset/historical/custom_reports/{reportId}/widgets/{widgetId}` | List historical custom report tabular data | `Listhistoricalcustomreporttabulardata` |
| GET | `/contact_center/analytics/dataset/historical/custom_reports/{reportId}/widgets/{widgetId}/schema` | Get historical custom report tabular schema | `getHistoricalCustomReportTabularSchema` |
| GET | `/contact_center/analytics/dataset/historical/disposition` | List historical disposition dataset data | `Listhistoricaldispositiondatasetdata` |
| GET | `/contact_center/analytics/dataset/historical/engagement` | List historical engagement dataset data | `Listengagementdatasetdata` |
| GET | `/contact_center/analytics/dataset/historical/engagement_timelines` | List historical engagement timelines | `listHistoricalEngagementTimelines` |
| GET | `/contact_center/analytics/dataset/historical/expert_assist` | List historical expert assist dataset data | `Listexpertassistdatasetdata` |
| GET | `/contact_center/analytics/dataset/historical/flow_performance` | List historical flow performance dataset data | `Listhistoricalflowperformancedatasetdata` |
| GET | `/contact_center/analytics/dataset/historical/outbound_dialer_performance` | List historical outbound dialer performance dataset data | `Listhistoricaloutbounddialerperformancedatasetdata` |
| GET | `/contact_center/analytics/dataset/historical/queue_performance` | List historical queue performance dataset data | `Listhistoricalqueueperformancedatasetdata` |
| GET | `/contact_center/analytics/log/historical/engagement` | List historical engagement log data | `Listhistoricalengagementlogs` |
| GET | `/contact_center/analytics/log/historical/journey` | List historical Zoom Phone to Contact Center call journey data | `ListhistoricalZoomphonetozcccalljourneydata` |
| GET | `/contact_center/reports/operation_logs` | List operation logs | `listOperationLogs` |

### Roles

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/roles` | List roles | `listRoles` |
| POST | `/contact_center/roles` | Create a role | `createRole` |
| POST | `/contact_center/roles/duplicate` | Duplicate a role | `Duplicatearole` |
| GET | `/contact_center/roles/{roleId}` | Get a role | `getRole` |
| DELETE | `/contact_center/roles/{roleId}` | Delete a role | `deleteRole` |
| PATCH | `/contact_center/roles/{roleId}` | Update a role | `updateRole` |
| DELETE | `/contact_center/roles/{roleId}/privileges` | Delete role privileges | `Deleteroleprivileges` |
| GET | `/contact_center/roles/{roleId}/users` | List users of a role | `getRoleUsers` |
| POST | `/contact_center/roles/{roleId}/users` | Assign a role | `assignRoleUsers` |
| DELETE | `/contact_center/roles/{roleId}/users/{userId}` | Unassign a user from a role | `deleteRoleUser` |

### Routing Profiles

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/agent_routing_profiles` | List agent routing profiles | `Listagentroutingprofiles` |
| POST | `/contact_center/agent_routing_profiles` | Create an agent routing profile | `Createanagentroutingprofile` |
| GET | `/contact_center/agent_routing_profiles/{agentRoutingProfileId}` | Get an agent routing profile | `getAgentRoutingProfile` |
| DELETE | `/contact_center/agent_routing_profiles/{agentRoutingProfileId}` | Delete an agent routing profile | `Deleteanagentroutingprofile` |
| PATCH | `/contact_center/agent_routing_profiles/{agentRoutingProfileId}` | Update an agent routing profile's details | `Updateanagentroutingprofile'sdetails` |
| GET | `/contact_center/consumer_routing_profiles` | List consumer routing profiles | `Listconsumerroutingprofiles` |
| POST | `/contact_center/consumer_routing_profiles` | Create a consumer routing profile | `Createaconsumerroutingprofile` |
| GET | `/contact_center/consumer_routing_profiles/{consumerRoutingProfileId}` | Get a consumer routing profile | `Getaconsumerroutingprofile` |
| DELETE | `/contact_center/consumer_routing_profiles/{consumerRoutingProfileId}` | Delete a consumer routing profile | `Deleteaconsumerroutingprofile` |
| PATCH | `/contact_center/consumer_routing_profiles/{consumerRoutingProfileId}` | Update a consumer routing profile's details | `Updateaconsumerroutingprofile'sdetails` |

### RTMS

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| PUT | `/contact_center/{engagementId}/rtms_app/status` | Update engagement Real-Time Media Streams (RTMS) app status | `engagementRTMSStatusUpdate` |

### Skills

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/skills` | List skills | `listSkills` |
| POST | `/contact_center/skills` | Create a skill | `skillCreate` |
| GET | `/contact_center/skills/categories` | List skill categories | `listSkillCategory` |
| POST | `/contact_center/skills/categories` | Create a skill category | `SkillCategoryCreate` |
| GET | `/contact_center/skills/categories/{skillCategoryId}` | Get a skill category | `getSkillCategory` |
| DELETE | `/contact_center/skills/categories/{skillCategoryId}` | Delete a skill category | `SkillCategoryDelete` |
| PATCH | `/contact_center/skills/categories/{skillCategoryId}` | Update a skill category | `SkillCategoryUpdate` |
| GET | `/contact_center/skills/{skillId}` | Get a skill | `getSkill` |
| DELETE | `/contact_center/skills/{skillId}` | Delete a skill | `skillDelete` |
| PATCH | `/contact_center/skills/{skillId}` | Update a skill | `skillNameUpdate` |
| GET | `/contact_center/skills/{skillId}/users` | List skill users | `listSkillUsers` |

### SMS

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/contact_center/sms` | Send an SMS | `contactCenterSMS` |

### Teams

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/teams` | List teams | `listTeams` |
| POST | `/contact_center/teams` | Create a team | `CreateTeam` |
| GET | `/contact_center/teams/{teamId}` | Get a team | `getTeamDetail` |
| DELETE | `/contact_center/teams/{teamId}` | Delete a team | `deleteTeam` |
| PATCH | `/contact_center/teams/{teamId}` | Update a team | `Updateateam` |
| GET | `/contact_center/teams/{teamId}/agents` | List team agents | `listTeamAgents` |
| POST | `/contact_center/teams/{teamId}/agents` | Assign team agents | `assignTeamAgents` |
| DELETE | `/contact_center/teams/{teamId}/agents` | Unassign team agents | `unassignTeamAgents` |
| GET | `/contact_center/teams/{teamId}/children` | List a team's child teams | `getTeamChildTeams` |
| PATCH | `/contact_center/teams/{teamId}/move` | Move a team | `moveTeam` |
| GET | `/contact_center/teams/{teamId}/parents` | List team's parent teams | `getTeamParentTeams` |
| GET | `/contact_center/teams/{teamId}/supervisors` | List team supervisors | `listTeamSupervisors` |
| POST | `/contact_center/teams/{teamId}/supervisors` | Assign team supervisors | `assignTeamSupervisors` |
| DELETE | `/contact_center/teams/{teamId}/supervisors` | Unassign team supervisors | `unassignTeamSupervisors` |

### Users

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/users` | List user profiles | `users` |
| POST | `/contact_center/users` | Create a user's profile | `createUser` |
| DELETE | `/contact_center/users` | Batch delete user profiles | `batchDeleteUsers` |
| PATCH | `/contact_center/users` | Batch update user profiles | `BatchUpdateUsers` |
| POST | `/contact_center/users/batch` | Batch create user profiles | `BatchCreateUsers` |
| PATCH | `/contact_center/users/status` | Batch update user status | `Batchupdateuserstatus` |
| GET | `/contact_center/users/templates` | List user templates | `ListUserTemplates` |
| POST | `/contact_center/users/templates` | Create a user template | `createAUserTemplate` |
| GET | `/contact_center/users/templates/{templateId}` | Get a user template | `Getanusertemplate` |
| DELETE | `/contact_center/users/templates/{templateId}` | Delete a user template | `deleteAUserTemplate` |
| PATCH | `/contact_center/users/templates/{templateId}` | Update a user template | `updateAUserTemplate` |
| GET | `/contact_center/users/{userId}` | Get a user's profile | `getUser` |
| DELETE | `/contact_center/users/{userId}` | Delete a user's profile | `userDelete` |
| PATCH | `/contact_center/users/{userId}` | Update a user's profile | `updateUser` |
| PATCH | `/contact_center/users/{userId}/opt_in_out_queues` | Opt-in or opt-out a user's queues | `BatchOptInOrOutUserQueues` |
| GET | `/contact_center/users/{userId}/queues` | List a user's queues | `listUserQueues` |
| GET | `/contact_center/users/{userId}/recordings` | List a user's recordings | `listUserRecordings` |
| DELETE | `/contact_center/users/{userId}/recordings` | Delete a user's recordings | `deleteUserRecordings` |
| GET | `/contact_center/users/{userId}/skills` | List a user's skills | `ListAUserSkills` |
| POST | `/contact_center/users/{userId}/skills` | Assign a user's skills | `assignSkills` |
| DELETE | `/contact_center/users/{userId}/skills/{skillId}` | Unassign a user's skill | `deleteASkill` |
| PATCH | `/contact_center/users/{userId}/status` | Update a user's status | `Updateauser'sstatus` |

### Variables

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/contact_center/variable_logs` | List variable logs | `listVariableLogs` |
| GET | `/contact_center/variable_logs/{variableLogId}` | Get a variable log | `getVariableLog` |
| DELETE | `/contact_center/variable_logs/{variableLogId}` | Delete a variable log | `deleteVariableLog` |
| GET | `/contact_center/variables` | List variables | `variables` |
| POST | `/contact_center/variables` | Create a variable | `createVariable` |
| GET | `/contact_center/variables/groups` | List variable groups | `listVariableGroups` |
| POST | `/contact_center/variables/groups` | Create a variable group | `createVariableGroup` |
| GET | `/contact_center/variables/groups/{variableGroupId}` | Get a variable group | `getAVariableGroup` |
| DELETE | `/contact_center/variables/groups/{variableGroupId}` | Delete a variable group | `DeleteGroup` |
| PATCH | `/contact_center/variables/groups/{variableGroupId}` | Update a variable group | `updateVariableGroup` |
| GET | `/contact_center/variables/{variableId}` | Get a variable | `variableGet` |
| DELETE | `/contact_center/variables/{variableId}` | Delete a variable | `variableDelete` |
| PATCH | `/contact_center/variables/{variableId}` | Update a variable | `variableUpdate` |
