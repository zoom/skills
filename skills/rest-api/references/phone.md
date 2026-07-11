# Zoom Phone API

Authoritative endpoint inventory for Phone. This file mirrors the official Zoom API Hub OpenAPI document for this product area.

## Canonical Source

- OpenAPI JSON: https://developers.zoom.us/api-hub/phone/methods/endpoints.json
- Base URL: `https://api.zoom.us/v2`
- Authentication details: [authentication.md](authentication.md)

## Notes

- Endpoint methods and paths below are generated from the official Zoom API Hub `paths` object.
- Scope names are defined per operation and frequently use granular scope names. Check the API Hub operation page for the exact scopes before implementation.
- Use this file for endpoint discovery and inventory. Use `../examples/` for orchestration patterns, not as the canonical source of path names.

## Coverage

| Metric | Value |
|--------|-------|
| Endpoint operations | 391 |
| Path templates | 238 |
| Tags | 47 |

## Tag Index

| Tag | Operations |
|-----|------------|
| Accounts | 4 |
| Alerts | 5 |
| Audio Library | 6 |
| Auto Receptionists | 19 |
| Billing Account | 2 |
| Blocked List | 5 |
| Call Handling | 4 |
| Call Logs | 15 |
| Call Queues | 32 |
| Carrier Reseller | 4 |
| Common Areas | 19 |
| Dashboard | 11 |
| Device Line Keys | 2 |
| Dial by Name Directory | 6 |
| Emergency Addresses | 5 |
| Emergency Service Locations | 6 |
| External Contacts | 5 |
| Fax | 8 |
| Firmware Update Rules | 6 |
| Group Call Pickup | 8 |
| Groups | 3 |
| Inbound Blocked List | 10 |
| IVR | 2 |
| Line Keys | 3 |
| Monitoring Groups | 9 |
| Outbound Calling | 24 |
| Phone Devices | 11 |
| Phone Numbers | 8 |
| Phone Plans | 2 |
| Phone Roles | 11 |
| Private Directory | 4 |
| Provider Exchange | 5 |
| Provision Templates | 5 |
| Recordings | 9 |
| Reports | 4 |
| Routing Rules | 5 |
| Setting Templates | 4 |
| Settings | 8 |
| Shared Line Appearance | 1 |
| Shared Line Group | 22 |
| Sites | 12 |
| SMS | 7 |
| SMS Campaign | 7 |
| SMS Consent | 1 |
| Users | 24 |
| Voicemails | 8 |
| Zoom Rooms | 10 |

## Endpoints by Tag

### Accounts

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/account_settings` | List an account's Zoom Phone settings | `listZoomPhoneAccountSettings` |
| GET | `/phone/outbound_caller_id/customized_numbers` | List an account's customized outbound caller ID phone numbers | `listCustomizeOutboundCallerNumbers` |
| POST | `/phone/outbound_caller_id/customized_numbers` | Add phone numbers for an account's customized outbound caller ID | `addOutboundCallerNumbers` |
| DELETE | `/phone/outbound_caller_id/customized_numbers` | Delete phone numbers for an account's customized outbound caller ID | `deleteOutboundCallerNumbers` |

### Alerts

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/alert_settings` | List alert settings with paging query | `ListAlertSettingsWithPagingQuery` |
| POST | `/phone/alert_settings` | Add an alert setting | `AddAnAlertSetting` |
| GET | `/phone/alert_settings/{alertSettingId}` | Get alert setting details | `GetAlertSettingDetails` |
| DELETE | `/phone/alert_settings/{alertSettingId}` | Delete an alert setting | `DeleteAnAlertSetting` |
| PATCH | `/phone/alert_settings/{alertSettingId}` | Update an alert setting | `UpdateAnAlertSetting` |

### Audio Library

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/audios/{audioId}` | Get an audio item | `GetAudioItem` |
| DELETE | `/phone/audios/{audioId}` | Delete an audio item | `DeleteAudioItem` |
| PATCH | `/phone/audios/{audioId}` | Update an audio item | `UpdateAudioItem` |
| GET | `/phone/users/{userId}/audios` | List audio items | `ListAudioItems` |
| POST | `/phone/users/{userId}/audios` | Add an audio item for text-to-speech conversion | `AddAnAudio` |
| POST | `/phone/users/{userId}/audios/batch` | Add audio items | `AddAudioItem` |

### Auto Receptionists

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/auto_receptionists` | List auto receptionists | `listAutoReceptionists` |
| POST | `/phone/auto_receptionists` | Add an auto receptionist | `addAutoReceptionist` |
| GET | `/phone/auto_receptionists/{autoReceptionistId}` | Get an auto receptionist | `getAutoReceptionistDetail` |
| DELETE | `/phone/auto_receptionists/{autoReceptionistId}` | Delete a non-primary auto receptionist | `deleteAutoReceptionist` |
| PATCH | `/phone/auto_receptionists/{autoReceptionistId}` | Update an auto receptionist | `updateAutoReceptionist` |
| GET | `/phone/auto_receptionists/{autoReceptionistId}/call_handling/settings` | Get auto receptionist call handling setting | `getAutoReceptionistCallHandlingSettings` |
| PATCH | `/phone/auto_receptionists/{autoReceptionistId}/call_handling/settings/{hourType}` | Update auto receptionist call handling setting | `updateAutoReceptionistCallHandlingSettings` |
| POST | `/phone/auto_receptionists/{autoReceptionistId}/phone_numbers` | Assign phone numbers | `assignPhoneNumbersAutoReceptionist` |
| DELETE | `/phone/auto_receptionists/{autoReceptionistId}/phone_numbers` | Unassign all phone numbers | `unassignAllPhoneNumsAutoReceptionist` |
| DELETE | `/phone/auto_receptionists/{autoReceptionistId}/phone_numbers/{phoneNumberId}` | Unassign a phone number | `unassignAPhoneNumAutoReceptionist` |
| GET | `/phone/auto_receptionists/{autoReceptionistId}/policies` | Get an auto receptionist policy | `getAutoReceptionistsPolicy` |
| PATCH | `/phone/auto_receptionists/{autoReceptionistId}/policies` | Update an auto receptionist policy | `updateAutoReceptionistPolicy` |
| POST | `/phone/auto_receptionists/{autoReceptionistId}/policies/{policyType}` | Add a policy subsetting | `AddPolicy` |
| DELETE | `/phone/auto_receptionists/{autoReceptionistId}/policies/{policyType}` | Delete a policy subsetting | `DeletePolicy` |
| PATCH | `/phone/auto_receptionists/{autoReceptionistId}/policies/{policyType}` | Update a policy subsetting | `updatePolicy` |
| GET | `/phone/auto_receptionists/{autoReceptionistId}/settings` | Get auto receptionist settings | `getAutoReceptionistSettings` |
| POST | `/phone/auto_receptionists/{autoReceptionistId}/settings/{settingType}` | Add auto receptionist setting | `addAutoReceptionistSetting` |
| DELETE | `/phone/auto_receptionists/{autoReceptionistId}/settings/{settingType}` | Delete auto receptionist setting | `deleteAutoReceptionistSetting` |
| PATCH | `/phone/auto_receptionists/{autoReceptionistId}/settings/{settingType}` | Update auto receptionist setting | `updateAutoReceptionistSetting` |

### Billing Account

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/billing_accounts` | List billing accounts | `listBillingAccount` |
| GET | `/phone/billing_accounts/{billingAccountId}` | Get billing account details | `GetABillingAccount` |

### Blocked List

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/blocked_list` | List blocked lists | `listBlockedList` |
| POST | `/phone/blocked_list` | Create a blocked list | `addAnumberToBlockedList` |
| GET | `/phone/blocked_list/{blockedListId}` | Get blocked list details | `getABlockedList` |
| DELETE | `/phone/blocked_list/{blockedListId}` | Delete a blocked list | `deleteABlockedList` |
| PATCH | `/phone/blocked_list/{blockedListId}` | Update a blocked list | `updateBlockedList` |

### Call Handling

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/extension/{extensionId}/call_handling/settings` | Get call handling settings | `getCallHandling` |
| POST | `/phone/extension/{extensionId}/call_handling/settings/{settingType}` | Add a call handling setting | `addCallHandling` |
| DELETE | `/phone/extension/{extensionId}/call_handling/settings/{settingType}` | Delete a call handling setting | `deleteCallHandling` |
| PATCH | `/phone/extension/{extensionId}/call_handling/settings/{settingType}` | Update a call handling setting | `updateCallHandling` |

### Call Logs

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/call_element/{callElementId}` | Get call element | `getCallElement` |
| GET | `/phone/call_history` | Get account's call history | `accountCallHistory` |
| GET | `/phone/call_history/{callHistoryUuid}` | Get call history | `getCallPath` |
| PATCH | `/phone/call_history/{callLogId}/client_code` | Add a client code to a call history | `addClientCodeToCallHistory` |
| GET | `/phone/call_history_detail/{callHistoryId}` | Get call history detail | `getCallHistoryDetail` |
| GET | `/phone/call_logs` | Get account's call logs | `accountCallLogs` |
| GET | `/phone/call_logs/{callLogId}` | Get call log details | `getCallLogDetails` |
| PUT | `/phone/call_logs/{callLogId}/client_code` | Add a client code to a call log | `addClientCodeToCallLog` |
| GET | `/phone/user/{userId}/ai_call_summary/{aiCallSummaryId}` | Get User AI Call Summary Detail | `getUserAICallSummary` |
| GET | `/phone/users/{userId}/call_history` | Get user's call history | `phoneUserCallHistory` |
| GET | `/phone/users/{userId}/call_history/sync` | Sync user's call history | `syncUserCallHistory` |
| DELETE | `/phone/users/{userId}/call_history/{callLogId}` | Delete a user's call history | `deleteUserCallHistory` |
| GET | `/phone/users/{userId}/call_logs` | Get user's call logs | `phoneUserCallLogs` |
| GET | `/phone/users/{userId}/call_logs/sync` | Sync user's call logs | `syncUserCallLogs` |
| DELETE | `/phone/users/{userId}/call_logs/{callLogId}` | Delete a user's call log | `deleteCallLog` |

### Call Queues

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/call_queue_analytics` | List call queue analytics | `callqueueanalytics` |
| GET | `/phone/call_queues` | List call queues | `listCallQueues` |
| POST | `/phone/call_queues` | Create a call queue | `createCallQueue` |
| GET | `/phone/call_queues/{callQueueId}` | Get call queue details | `getACallQueue` |
| DELETE | `/phone/call_queues/{callQueueId}` | Delete a call queue | `deleteACallQueue` |
| PATCH | `/phone/call_queues/{callQueueId}` | Update call queue details | `updateCallQueue` |
| GET | `/phone/call_queues/{callQueueId}/call_handling/settings` | Get call queue call handling setting | `getCallQueueCallHandlingSetting` |
| PATCH | `/phone/call_queues/{callQueueId}/call_handling/settings/{hourType}` | Update call queue call handling setting | `updateCallQueueCallHandlingSetting` |
| GET | `/phone/call_queues/{callQueueId}/custom_groups` | List custom groups of a call queue | `Listcustomgroupsofacallqueue` |
| POST | `/phone/call_queues/{callQueueId}/custom_groups` | Create a custom group in a call queue | `Createacustomgroupinacallqueue` |
| GET | `/phone/call_queues/{callQueueId}/custom_groups/{customGroupId}` | Get call queue custom group detail | `Getacustomgroup` |
| DELETE | `/phone/call_queues/{callQueueId}/custom_groups/{customGroupId}` | Delete a custom group | `Deleteacustomgroup` |
| PATCH | `/phone/call_queues/{callQueueId}/custom_groups/{customGroupId}` | Update a custom group | `Updateacustomgroup` |
| POST | `/phone/call_queues/{callQueueId}/custom_groups/{customGroupId}/members` | Add members to a custom group | `Addmemberstoacustomgroup` |
| DELETE | `/phone/call_queues/{callQueueId}/custom_groups/{customGroupId}/members/{extensionId}` | Remove a member from a custom group | `Removeamemberfromacustomgroup` |
| GET | `/phone/call_queues/{callQueueId}/members` | List call queue members | `listCallQueueMembers` |
| POST | `/phone/call_queues/{callQueueId}/members` | Add members to a call queue | `addMembersToCallQueue` |
| DELETE | `/phone/call_queues/{callQueueId}/members` | Unassign all members | `unassignAllMembers` |
| DELETE | `/phone/call_queues/{callQueueId}/members/{memberId}` | Unassign a member | `unassignMemberFromCallQueue` |
| POST | `/phone/call_queues/{callQueueId}/phone_numbers` | Assign numbers to a call queue | `assignPhoneToCallQueue` |
| DELETE | `/phone/call_queues/{callQueueId}/phone_numbers` | Unassign all phone numbers | `unassignAPhoneNumCallQueue` |
| DELETE | `/phone/call_queues/{callQueueId}/phone_numbers/{phoneNumberId}` | Unassign a phone number | `unAssignPhoneNumCallQueue` |
| GET | `/phone/call_queues/{callQueueId}/policies` | Get call queue policy | `getCallQueuePolicy` |
| PATCH | `/phone/call_queues/{callQueueId}/policies` | Update call queue policy | `updateCallQueuePolicy` |
| POST | `/phone/call_queues/{callQueueId}/policies/{policyType}` | Add a policy subsetting to a call queue | `addCQPolicySubSetting` |
| DELETE | `/phone/call_queues/{callQueueId}/policies/{policyType}` | Delete a CQ policy setting | `removeCQPolicySubSetting` |
| PATCH | `/phone/call_queues/{callQueueId}/policies/{policyType}` | Update a call queue's policy subsetting | `updateCQPolicySubSetting` |
| GET | `/phone/call_queues/{callQueueId}/recordings` | Get call queue recordings | `getCallQueueRecordings` |
| GET | `/phone/call_queues/{callQueueId}/settings` | Get call queue settings | `getCallQueueSettings` |
| POST | `/phone/call_queues/{callQueueId}/settings/{settingType}` | Add call queue setting | `addCallQueueSetting` |
| DELETE | `/phone/call_queues/{callQueueId}/settings/{settingType}` | Delete call queue setting | `deleteCallQueueSetting` |
| PATCH | `/phone/call_queues/{callQueueId}/settings/{settingType}` | Update call queue setting | `updateCallQueueSetting` |

### Carrier Reseller

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/carrier_reseller/numbers` | List phone numbers | `listCRPhoneNumbers` |
| POST | `/phone/carrier_reseller/numbers` | Create phone numbers | `createCRPhoneNumbers` |
| PATCH | `/phone/carrier_reseller/numbers` | Activate phone numbers | `activeCRPhoneNumbers` |
| DELETE | `/phone/carrier_reseller/numbers/{number}` | Delete a phone number | `deleteCRPhoneNumber` |

### Common Areas

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/common_areas` | List common areas | `listCommonAreas` |
| POST | `/phone/common_areas` | Add a common area | `addCommonArea` |
| POST | `/phone/common_areas/activation_code` | Generate activation codes for common areas | `Generateactivationcodesforcommonareas` |
| GET | `/phone/common_areas/activation_codes` | List activation codes | `listActivationCodes` |
| POST | `/phone/common_areas/template_id/{templateId}` | Apply template to common areas | `ApplyTemplatetoCommonAreas` |
| GET | `/phone/common_areas/{commonAreaId}` | Get common area details | `getACommonArea` |
| DELETE | `/phone/common_areas/{commonAreaId}` | Delete a common area | `deleteCommonArea` |
| PATCH | `/phone/common_areas/{commonAreaId}` | Update common area | `updateCommonArea` |
| GET | `/phone/common_areas/{commonAreaId}/call_handling/settings` | Get common area call handling setting | `getCommonAreaCallHandlingSetting` |
| PATCH | `/phone/common_areas/{commonAreaId}/call_handling/settings/{hourType}` | Update common area call handling setting | `updateCommonAreaCallHandlingSetting` |
| POST | `/phone/common_areas/{commonAreaId}/calling_plans` | Assign calling plans to a common area | `assignCallingPlansToCommonArea` |
| DELETE | `/phone/common_areas/{commonAreaId}/calling_plans/{type}` | Unassign a calling plan from the common area | `unassignCallingPlansFromCommonArea` |
| POST | `/phone/common_areas/{commonAreaId}/phone_numbers` | Assign phone numbers to a common area | `assignPhoneNumbersToCommonArea` |
| DELETE | `/phone/common_areas/{commonAreaId}/phone_numbers/{phoneNumberId}` | Unassign phone numbers from common area | `unassignPhoneNumbersFromCommonArea` |
| PATCH | `/phone/common_areas/{commonAreaId}/pin_code` | Update common area pin code | `UpdateCommonAreaPinCode` |
| GET | `/phone/common_areas/{commonAreaId}/settings` | Get common area settings | `getCommonAreaSettings` |
| POST | `/phone/common_areas/{commonAreaId}/settings/{settingType}` | Add common area setting | `AddCommonAreaSetting` |
| DELETE | `/phone/common_areas/{commonAreaId}/settings/{settingType}` | Delete common area setting | `deleteCommonAreaSetting` |
| PATCH | `/phone/common_areas/{commonAreaId}/settings/{settingType}` | Update common area setting | `UpdateCommonAreaSetting` |

### Dashboard

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/metrics/call_logs` | List call logs | `listCallLogsMetrics` |
| GET | `/phone/metrics/call_logs/{callId}/qos` | Get call QoS | `getCallQoS` |
| GET | `/phone/metrics/call_logs/{call_id}` | Get call details from call log | `getCallLogMetricsDetails` |
| GET | `/phone/metrics/emergency_services/default_emergency_address/users` | List default emergency address users | `listUserDefaultEmergencyAddress` |
| GET | `/phone/metrics/emergency_services/detectable_personal_location/users` | List detectable personal location users | `listUserDetectablePersonalLocation` |
| GET | `/phone/metrics/emergency_services/location_sharing_permission/users` | List users permission for location sharing | `listUserLocationSharingPermission` |
| GET | `/phone/metrics/emergency_services/nomadic_emergency_services/users` | List nomadic emergency services users | `listUserNomadicEmergencyServices` |
| GET | `/phone/metrics/emergency_services/realtime_location/devices` | List real time location for IP phones | `listPhoneRealtimelocation` |
| GET | `/phone/metrics/emergency_services/realtime_location/users` | List real time location for users | `listUserRealtimeLocation` |
| GET | `/phone/metrics/location_tracking` | List tracked locations | `listTrackedLocations` |
| GET | `/phone/metrics/past_calls` | List past call metrics | `listPastCallMetrics` |

### Device Line Keys

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/devices/{deviceId}/line_keys` | Get device line keys information | `listDeviceLineKeySetting` |
| PATCH | `/phone/devices/{deviceId}/line_keys` | Batch update device line key position | `batchUpdateDeviceLineKeySetting` |

### Dial by Name Directory

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/dial_by_name_directory/extensions` | List users in directory | `ListUsersFromDirectory` |
| POST | `/phone/dial_by_name_directory/extensions` | Add users to a directory | `AddUsersToDirectory` |
| DELETE | `/phone/dial_by_name_directory/extensions` | Delete users from a directory | `DeleteUsersFromDirectory` |
| GET | `/phone/sites/{siteId}/dial_by_name_directory/extensions` | List users in a directory by site | `ListUsersFromDirectoryBySite` |
| POST | `/phone/sites/{siteId}/dial_by_name_directory/extensions` | Add users to a directory of a site | `AddUsersToDirectoryBySite` |
| DELETE | `/phone/sites/{siteId}/dial_by_name_directory/extensions` | Delete users from a directory of a site | `DeleteUsersFromDirectoryBySite` |

### Emergency Addresses

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/emergency_addresses` | List emergency addresses | `listEmergencyAddresses` |
| POST | `/phone/emergency_addresses` | Add an emergency address | `addEmergencyAddress` |
| GET | `/phone/emergency_addresses/{emergencyAddressId}` | Get emergency address details | `getEmergencyAddress` |
| DELETE | `/phone/emergency_addresses/{emergencyAddressId}` | Delete an emergency address | `deleteEmergencyAddress` |
| PATCH | `/phone/emergency_addresses/{emergencyAddressId}` | Update an emergency address | `updateEmergencyAddress` |

### Emergency Service Locations

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/phone/batch_locations` | Batch add emergency service locations | `batchAddLocations` |
| GET | `/phone/locations` | List emergency service locations | `listLocations` |
| POST | `/phone/locations` | Add an emergency service location | `addLocation` |
| GET | `/phone/locations/{locationId}` | Get emergency service location details | `getLocation` |
| DELETE | `/phone/locations/{locationId}` | Delete an emergency location | `deleteLocation` |
| PATCH | `/phone/locations/{locationId}` | Update emergency service location | `updateLocation` |

### External Contacts

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/external_contacts` | List external contacts | `listExternalContacts` |
| POST | `/phone/external_contacts` | Add an external contact | `addExternalContact` |
| GET | `/phone/external_contacts/{externalContactId}` | Get external contact details | `getAExternalContact` |
| DELETE | `/phone/external_contacts/{externalContactId}` | Delete an external contact | `deleteAExternalContact` |
| PATCH | `/phone/external_contacts/{externalContactId}` | Update external contact | `updateExternalContact` |

### Fax

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/fax/files` | Upload fax file | `UploadFaxFiles` |
| GET | `/phone/extension/{extensionId}/fax/logs` | Get extension's fax logs | `Getuser'sfaxlogs` |
| POST | `/phone/fax/documents` | Send fax | `SendEFax` |
| GET | `/phone/fax/logs` | Get account's fax logs | `GetAccount'sFaxLogs` |
| GET | `/phone/fax/logs/{faxLogId}` | Get fax log details | `GetFaxLogDetails` |
| DELETE | `/phone/fax/logs/{faxLogId}` | Delete fax log | `DeleteFaxLog` |
| PATCH | `/phone/fax/logs/{faxLogId}` | Update fax log read status | `UpdateFaxLogReadStatus` |
| GET | `/phone/fax/logs/{faxLogId}/file/{fileId}` | Download fax file | `Downloadfaxfile` |

### Firmware Update Rules

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/firmware_update_rules` | List firmware update rules | `ListFirmwareRules` |
| POST | `/phone/firmware_update_rules` | Add a firmware update rule | `AddFirmwareRule` |
| GET | `/phone/firmware_update_rules/{ruleId}` | Get firmware update rule information | `GetFirmwareRuleDetail` |
| DELETE | `/phone/firmware_update_rules/{ruleId}` | Delete firmware update rule | `DeleteFirmwareUpdateRule` |
| PATCH | `/phone/firmware_update_rules/{ruleId}` | Update firmware update rule | `UpdateFirmwareRule` |
| GET | `/phone/firmwares` | List updatable firmwares | `ListFirmwares` |

### Group Call Pickup

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/group_call_pickup` | List group call pickup objects | `listGCP` |
| POST | `/phone/group_call_pickup` | Add a group call pickup object | `addGCP` |
| GET | `/phone/group_call_pickup/{groupId}` | Get call pickup group by ID | `GetGCP` |
| DELETE | `/phone/group_call_pickup/{groupId}` | Delete group call pickup objects | `deleteGCP` |
| PATCH | `/phone/group_call_pickup/{groupId}` | Update the group call pickup information | `updateGCP` |
| GET | `/phone/group_call_pickup/{groupId}/members` | List call pickup group members | `listGCPMembers` |
| POST | `/phone/group_call_pickup/{groupId}/members` | Add members to a call pickup group | `addGCPMembers` |
| DELETE | `/phone/group_call_pickup/{groupId}/members/{extensionId}` | Remove members from call pickup group | `removeGCPMembers` |

### Groups

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/groups/{groupId}/policies/{policyType}` | Get group policy details | `GetGroupPolicyDetails` |
| PATCH | `/phone/groups/{groupId}/policies/{policyType}` | Update group policy | `updateGroupPolicy` |
| GET | `/phone/groups/{groupId}/settings` | Get group phone settings | `getGroupPhoneSettings` |

### Inbound Blocked List

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/extension/{extensionId}/inbound_blocked/rules` | List an extension's inbound block rules | `ListExtensionLevelInboundBlockRules` |
| POST | `/phone/extension/{extensionId}/inbound_blocked/rules` | Add an extension's inbound block rule | `AddExtensiontLevelInboundBlockRules` |
| DELETE | `/phone/extension/{extensionId}/inbound_blocked/rules` | Delete an extension's inbound block rule | `DeleteExtensiontLevelInboundBlockRules` |
| GET | `/phone/inbound_blocked/extension_rules/statistics` | List an account's inbound blocked statistics | `ListAccountLevelInboundBlockedStatistics` |
| DELETE | `/phone/inbound_blocked/extension_rules/statistics` | Delete an account's inbound blocked statistics | `DeleteAccountLevelInboundBlockedStatistics` |
| PATCH | `/phone/inbound_blocked/extension_rules/statistics/blocked_for_all` | Mark a phone number as blocked for all extensions | `MarkPhoneNumberAsBlockedForAllExtensions` |
| GET | `/phone/inbound_blocked/rules` | List an account's inbound block rules | `ListAccountLevelInboundBlockRules` |
| POST | `/phone/inbound_blocked/rules` | Add an account's inbound block rule | `AddAccountLevelInboundBlockRules` |
| DELETE | `/phone/inbound_blocked/rules` | Delete an account's inbound block rule | `DeleteAccountLevelInboundBlockRules` |
| PATCH | `/phone/inbound_blocked/rules/{blockedRuleId}` | Update an account's inbound block rule | `UpdateAccountLevelInboundBlockRule` |

### IVR

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/auto_receptionists/{autoReceptionistId}/ivr` | Get auto receptionist IVR | `getAutoReceptionistIVR` |
| PATCH | `/phone/auto_receptionists/{autoReceptionistId}/ivr` | Update auto receptionist IVR | `updateAutoReceptionistIVR` |

### Line Keys

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/extension/{extensionId}/line_keys` | Get line key position and settings information | `listLineKeySetting` |
| PATCH | `/phone/extension/{extensionId}/line_keys` | Batch update line key position and settings information | `BatchUpdateLineKeySetting` |
| DELETE | `/phone/extension/{extensionId}/line_keys/{lineKeyId}` | Delete a line key setting. | `DeleteLineKey` |

### Monitoring Groups

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/monitoring_groups` | Get a list of monitoring groups on an account | `listMonitoringGroup` |
| POST | `/phone/monitoring_groups` | Create a monitoring group | `createMonitoringGroup` |
| GET | `/phone/monitoring_groups/{monitoringGroupId}` | Get monitoring group by ID | `getMonitoringGroupById` |
| DELETE | `/phone/monitoring_groups/{monitoringGroupId}` | Delete a monitoring group | `deleteMonitoringGroup` |
| PATCH | `/phone/monitoring_groups/{monitoringGroupId}` | Update a monitoring group | `updateMonitoringGroup` |
| GET | `/phone/monitoring_groups/{monitoringGroupId}/monitor_members` | Get members of a monitoring group | `listMembers` |
| POST | `/phone/monitoring_groups/{monitoringGroupId}/monitor_members` | Add members to a monitoring group | `addMembers` |
| DELETE | `/phone/monitoring_groups/{monitoringGroupId}/monitor_members` | Remove all monitors or monitored members from a monitoring group | `removeMembers` |
| DELETE | `/phone/monitoring_groups/{monitoringGroupId}/monitor_members/{memberExtensionId}` | Remove a member from a monitoring group | `removeMember` |

### Outbound Calling

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/common_areas/{commonAreaId}/outbound_calling/countries_regions` | Get common area level outbound calling countries and regions | `GetCommonAreaOutboundCallingCountriesAndRegions` |
| PATCH | `/phone/common_areas/{commonAreaId}/outbound_calling/countries_regions` | Update common area level outbound calling countries or regions | `UpdateCommonAreaOutboundCallingCountriesOrRegions` |
| GET | `/phone/common_areas/{commonAreaId}/outbound_calling/exception_rules` | List common area level outbound calling exception rules | `listCommonAreaOutboundCallingExceptionRule` |
| POST | `/phone/common_areas/{commonAreaId}/outbound_calling/exception_rules` | Add common area level outbound calling exception rule | `AddCommonAreaOutboundCallingExceptionRule` |
| DELETE | `/phone/common_areas/{commonAreaId}/outbound_calling/exception_rules/{exceptionRuleId}` | Delete common area level outbound calling exception rule | `deleteCommonAreaOutboundCallingExceptionRule` |
| PATCH | `/phone/common_areas/{commonAreaId}/outbound_calling/exception_rules/{exceptionRuleId}` | Update common area level outbound calling exception rule | `UpdateCommonAreaOutboundCallingExceptionRule` |
| GET | `/phone/outbound_calling/countries_regions` | Get account level outbound calling countries and regions | `GetAccountOutboundCallingCountriesAndRegions` |
| PATCH | `/phone/outbound_calling/countries_regions` | Update account level outbound calling countries or regions | `UpdateAccountOutboundCallingCountriesOrRegions` |
| GET | `/phone/outbound_calling/exception_rules` | List account level outbound calling exception rules | `listAccountOutboundCallingExceptionRule` |
| POST | `/phone/outbound_calling/exception_rules` | Add account level outbound calling exception rule | `AddAccountOutboundCallingExceptionRule` |
| DELETE | `/phone/outbound_calling/exception_rules/{exceptionRuleId}` | Delete account level outbound calling exception rule | `deleteAccountOutboundCallingExceptionRule` |
| PATCH | `/phone/outbound_calling/exception_rules/{exceptionRuleId}` | Update account level outbound calling exception rule | `UpdateAccountOutboundCallingExceptionRule` |
| GET | `/phone/sites/{siteId}/outbound_calling/countries_regions` | Get site level outbound calling countries and regions | `GetSiteOutboundCallingCountriesAndRegions` |
| PATCH | `/phone/sites/{siteId}/outbound_calling/countries_regions` | Update site level outbound calling countries or regions | `UpdateSiteOutboundCallingCountriesOrRegions` |
| GET | `/phone/sites/{siteId}/outbound_calling/exception_rules` | List site level outbound calling exception rules | `listSiteOutboundCallingExceptionRule` |
| POST | `/phone/sites/{siteId}/outbound_calling/exception_rules` | Add site level outbound calling exception rule | `AddSiteOutboundCallingExceptionRule` |
| DELETE | `/phone/sites/{siteId}/outbound_calling/exception_rules/{exceptionRuleId}` | Delete site level outbound calling exception rule | `deleteSiteOutboundCallingExceptionRule` |
| PATCH | `/phone/sites/{siteId}/outbound_calling/exception_rules/{exceptionRuleId}` | Update site level outbound calling exception rule | `UpdateSiteOutboundCallingExceptionRule` |
| GET | `/phone/users/{userId}/outbound_calling/countries_regions` | Get user level outbound calling countries and regions | `GetUserOutboundCallingCountriesAndRegions` |
| PATCH | `/phone/users/{userId}/outbound_calling/countries_regions` | Update user level outbound calling countries or regions | `UpdateUserOutboundCallingCountriesOrRegions` |
| GET | `/phone/users/{userId}/outbound_calling/exception_rules` | List user level outbound calling exception rules | `listUserOutboundCallingExceptionRule` |
| POST | `/phone/users/{userId}/outbound_calling/exception_rules` | Add user level outbound calling exception rule | `AddUserOutboundCallingExceptionRule` |
| DELETE | `/phone/users/{userId}/outbound_calling/exception_rules/{exceptionRuleId}` | Delete user level outbound calling exception rule | `deleteUserOutboundCallingExceptionRule` |
| PATCH | `/phone/users/{userId}/outbound_calling/exception_rules/{exceptionRuleId}` | Update user level outbound calling exception rule | `UpdateUserOutboundCallingExceptionRule` |

### Phone Devices

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/devices` | List devices | `listPhoneDevices` |
| POST | `/phone/devices` | Add a device | `addPhoneDevice` |
| POST | `/phone/devices/sync` | Sync deskphones | `syncPhoneDevice` |
| GET | `/phone/devices/{deviceId}` | Get device details | `getADevice` |
| DELETE | `/phone/devices/{deviceId}` | Delete a device | `deleteADevice` |
| PATCH | `/phone/devices/{deviceId}` | Update a device | `updateADevice` |
| POST | `/phone/devices/{deviceId}/extensions` | Assign an entity to a device | `addExtensionsToADevice` |
| DELETE | `/phone/devices/{deviceId}/extensions/{extensionId}` | Unassign an entity from the device | `deleteExtensionFromADevice` |
| PUT | `/phone/devices/{deviceId}/provision_templates` | Update provision template of a device | `updateProvisionTemplateToDevice` |
| POST | `/phone/devices/{deviceId}/reboot` | Reboot a desk phone | `rebootPhoneDevice` |
| GET | `/phone/smartphones` | List Smartphones | `ListSmartphones` |

### Phone Numbers

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/phone/byoc_numbers` | Add BYOC phone numbers | `addBYOCNumber` |
| GET | `/phone/numbers` | List phone numbers | `listAccountPhoneNumbers` |
| DELETE | `/phone/numbers` | Delete unassigned phone numbers | `deleteUnassignedPhoneNumbers` |
| PATCH | `/phone/numbers/sites/{siteId}` | Update a site's unassigned phone numbers | `updateSiteForUnassignedPhoneNumbers` |
| GET | `/phone/numbers/{phoneNumberId}` | Get a phone number | `getPhoneNumberDetails` |
| PATCH | `/phone/numbers/{phoneNumberId}` | Update a phone number | `updatePhoneNumberDetails` |
| POST | `/phone/users/{userId}/phone_numbers` | Assign a phone number to a user | `assignPhoneNumber` |
| DELETE | `/phone/users/{userId}/phone_numbers/{phoneNumberId}` | Unassign a phone number | `UnassignPhoneNumber` |

### Phone Plans

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/calling_plans` | List calling plans | `listCallingPlans` |
| GET | `/phone/plans` | List plan information | `listPhonePlans` |

### Phone Roles

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/roles` | List phone roles | `ListPhoneRoles` |
| POST | `/phone/roles` | Duplicate a phone role | `DuplicatePhoneRole` |
| GET | `/phone/roles/{roleId}` | Get role information | `getRoleInformation` |
| DELETE | `/phone/roles/{roleId}` | Delete a phone role | `DeletePhoneRole` |
| PATCH | `/phone/roles/{roleId}` | Update a phone role | `UpdatePhoneRole` |
| GET | `/phone/roles/{roleId}/members` | List members in a role | `ListRoleMembers` |
| POST | `/phone/roles/{roleId}/members` | Add members to roles | `AddRoleMembers` |
| DELETE | `/phone/roles/{roleId}/members` | Delete members in a role | `DelRoleMembers` |
| GET | `/phone/roles/{roleId}/targets` | List phone role targets | `ListPhoneRoleTargets` |
| POST | `/phone/roles/{roleId}/targets` | Add phone role targets | `AddPhoneRoleTargets` |
| DELETE | `/phone/roles/{roleId}/targets` | Delete phone role targets | `DeletePhoneRoleTargets` |

### Private Directory

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/private_directory/members` | List private directory members | `listPrivateDirectoryMembers` |
| POST | `/phone/private_directory/members` | Add members to a private directory | `addMembersToAPrivateDirectory` |
| DELETE | `/phone/private_directory/members/{extensionId}` | Remove a member from a private directory | `removeAMemberFromAPrivateDirectory` |
| PATCH | `/phone/private_directory/members/{extensionId}` | Update a private directory member | `updateAPrivateDirectoryMember` |

### Provider Exchange

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/carrier_peering/numbers` | List carrier peering phone numbers. | `listCarrierPeeringPhoneNumbers` |
| GET | `/phone/peering/numbers` | List peering phone numbers | `listPeeringPhoneNumbers` |
| POST | `/phone/peering/numbers` | Add peering phone numbers | `addPeeringPhoneNumbers` |
| DELETE | `/phone/peering/numbers` | Remove peering phone numbers | `deletePeeringPhoneNumbers` |
| PATCH | `/phone/peering/numbers` | Update peering phone numbers | `updatePeeringPhoneNumbers` |

### Provision Templates

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/provision_templates` | List provision templates | `listAccountProvisionTemplate` |
| POST | `/phone/provision_templates` | Add a provision template | `addProvisionTemplate` |
| GET | `/phone/provision_templates/{templateId}` | Get a provision template | `GetProvisionTemplate` |
| DELETE | `/phone/provision_templates/{templateId}` | Delete a provision template | `deleteProvisionTemplate` |
| PATCH | `/phone/provision_templates/{templateId}` | Update a provision template | `updateProvisionTemplate` |

### Recordings

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/call_element/{callElementId}/recording` | Get a recording by call element ID | `getPhoneRecordingByCallElementId` |
| GET | `/phone/call_logs/{id}/recordings` | Get recording by call ID | `getPhoneRecordingsByCallIdOrCallLogId` |
| GET | `/phone/recording/download/{fileId}` | Download a phone recording | `phoneDownloadRecordingFile` |
| GET | `/phone/recording_transcript/download/{recordingId}` | Download a phone recording transcript | `phoneDownloadRecordingTranscript` |
| GET | `/phone/recordings` | Get call recordings | `getPhoneRecordings` |
| DELETE | `/phone/recordings/{recordingId}` | Delete a call recording | `deleteCallRecording` |
| PATCH | `/phone/recordings/{recordingId}` | Update auto delete field | `UpdateAutoDeleteField` |
| PUT | `/phone/recordings/{recordingId}/status` | Update Recording Status | `UpdateRecordingStatus` |
| GET | `/phone/users/{userId}/recordings` | Get user's recordings | `phoneUserRecordings` |

### Reports

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/reports/call_charges` | Get call charges usage report | `GetCallChargesUsageReport` |
| GET | `/phone/reports/fax_charges` | Get fax charges usage report | `Getfaxchargesusagereport` |
| GET | `/phone/reports/operationlogs` | Get operation logs report | `getPSOperationLogs` |
| GET | `/phone/reports/sms_charges` | Get SMS/MMS charges usage report | `GetSMSChargesUsageReport` |

### Routing Rules

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/routing_rules` | List directory backup routing rules | `listRoutingRule` |
| POST | `/phone/routing_rules` | Add directory backup routing rule | `addRoutingRule` |
| GET | `/phone/routing_rules/{routingRuleId}` | Get directory backup routing rule | `getRoutingRule` |
| DELETE | `/phone/routing_rules/{routingRuleId}` | Delete directory backup routing rule | `deleteRoutingRule` |
| PATCH | `/phone/routing_rules/{routingRuleId}` | Update directory backup routing rule | `updateRoutingRule` |

### Setting Templates

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/setting_templates` | List setting templates | `listSettingTemplates` |
| POST | `/phone/setting_templates` | Add a setting template | `addSettingTemplate` |
| GET | `/phone/setting_templates/{templateId}` | Get setting template details | `getSettingTemplate` |
| PATCH | `/phone/setting_templates/{templateId}` | Update a setting template | `updateSettingTemplate` |

### Settings

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/policies/{policyType}` | Get account policy details | `GetAccountPolicyDetails` |
| PATCH | `/phone/policies/{policyType}` | Update account policy | `updateAccountPolicy` |
| GET | `/phone/ported_numbers/orders` | List ported numbers | `listPortedNumbers` |
| GET | `/phone/ported_numbers/orders/{orderId}` | Get ported numbers details | `getPortedNumbersDetails` |
| GET | `/phone/settings` | Get phone account settings | `phoneSetting` |
| PATCH | `/phone/settings` | Update phone account settings | `updatePhoneSettings` |
| GET | `/phone/sip_groups` | List SIP groups | `listSipGroups` |
| GET | `/phone/sip_trunk/trunks` | List BYOC SIP trunks | `listBYOCSIPTrunk` |

### Shared Line Appearance

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/shared_line_appearances` | List shared line appearances | `listSharedLineAppearances` |

### Shared Line Group

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/shared_line_groups` | List shared line groups | `listSharedLineGroups` |
| POST | `/phone/shared_line_groups` | Create a shared line group | `createASharedLineGroup` |
| GET | `/phone/shared_line_groups/{sharedLineGroupId}` | Get a shared line group | `getASharedLineGroup` |
| GET | `/phone/shared_line_groups/{sharedLineGroupId}/call_handling/settings` | Get shared line group call handling setting | `getSharedLineGroupCallHandlingSetting` |
| PATCH | `/phone/shared_line_groups/{sharedLineGroupId}/call_handling/settings/{hourType}` | Update shared line group call handling setting | `updateSharedLineGroupCallHandlingSetting` |
| GET | `/phone/shared_line_groups/{sharedLineGroupId}/policies` | Get a shared line group policy | `getSharedLineGroupPolicy` |
| PATCH | `/phone/shared_line_groups/{sharedLineGroupId}/policies` | Update a shared line group policy | `updateSharedLineGroupPolicy` |
| GET | `/phone/shared_line_groups/{sharedLineGroupId}/settings` | Get shared line group settings | `getSharedLineGroupSettings` |
| POST | `/phone/shared_line_groups/{sharedLineGroupId}/settings/{settingType}` | Add shared line group setting | `addSharedLineGroupSetting` |
| DELETE | `/phone/shared_line_groups/{sharedLineGroupId}/settings/{settingType}` | Delete shared line group setting | `deleteSharedLineGroupSetting` |
| PATCH | `/phone/shared_line_groups/{sharedLineGroupId}/settings/{settingType}` | Update shared line group setting | `updateSharedLineGroupSetting` |
| DELETE | `/phone/shared_line_groups/{slgId}` | Delete a shared line group | `deleteASharedLineGroup` |
| PATCH | `/phone/shared_line_groups/{slgId}` | Update a shared line group | `updateASharedLineGroup` |
| POST | `/phone/shared_line_groups/{slgId}/members` | Add members to a shared line group | `addMembersToSharedLineGroup` |
| DELETE | `/phone/shared_line_groups/{slgId}/members` | Unassign members from a shared line group | `deleteMembersOfSLG` |
| DELETE | `/phone/shared_line_groups/{slgId}/members/{memberId}` | Unassign a member from a shared line group | `deleteAMemberSLG` |
| POST | `/phone/shared_line_groups/{slgId}/phone_numbers` | Assign phone numbers | `assignPhoneNumbersSLG` |
| DELETE | `/phone/shared_line_groups/{slgId}/phone_numbers` | Unassign all phone numbers | `deletePhoneNumbersSLG` |
| DELETE | `/phone/shared_line_groups/{slgId}/phone_numbers/{phoneNumberId}` | Unassign a phone number | `deleteAPhoneNumberSLG` |
| POST | `/phone/shared_line_groups/{slgId}/policies/{policyType}` | Add a policy setting to a shared line group | `addSLGPolicySubSetting` |
| DELETE | `/phone/shared_line_groups/{slgId}/policies/{policyType}` | Delete an SLG policy setting | `removeSLGPolicySubSetting` |
| PATCH | `/phone/shared_line_groups/{slgId}/policies/{policyType}` | Update an SLG policy setting | `updateSLGPolicySubSetting` |

### Sites

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/sites` | List phone sites | `listPhoneSites` |
| POST | `/phone/sites` | Create a phone site | `createPhoneSite` |
| GET | `/phone/sites/{siteId}` | Get phone site details | `getASite` |
| DELETE | `/phone/sites/{siteId}` | Delete a phone site | `deletePhoneSite` |
| PATCH | `/phone/sites/{siteId}` | Update phone site details | `updateSiteDetails` |
| GET | `/phone/sites/{siteId}/outbound_caller_id/customized_numbers` | List customized outbound caller ID phone numbers | `listSiteCustomizeOutboundCallerNumbers` |
| POST | `/phone/sites/{siteId}/outbound_caller_id/customized_numbers` | Add customized outbound caller ID phone numbers | `addSiteOutboundCallerNumbers` |
| DELETE | `/phone/sites/{siteId}/outbound_caller_id/customized_numbers` | Remove customized outbound caller ID phone numbers | `deleteSiteOutboundCallerNumbers` |
| GET | `/phone/sites/{siteId}/settings/{settingType}` | Get a phone site setting | `getSiteSettingForType` |
| POST | `/phone/sites/{siteId}/settings/{settingType}` | Add a site setting | `addSiteSetting` |
| DELETE | `/phone/sites/{siteId}/settings/{settingType}` | Delete a site setting | `deleteSiteSetting` |
| PATCH | `/phone/sites/{siteId}/settings/{settingType}` | Update the site setting | `updateSiteSetting` |

### SMS

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/phone/sms/messages` | Post SMS message | `postSmsMessage` |
| GET | `/phone/sms/sessions` | Get account's SMS sessions | `accountSmsSession` |
| GET | `/phone/sms/sessions/{sessionId}` | Get SMS session details | `smsSessionDetails` |
| GET | `/phone/sms/sessions/{sessionId}/messages/{messageId}` | Get SMS by message ID | `smsByMessageId` |
| GET | `/phone/sms/sessions/{sessionId}/sync` | Sync SMS by session ID | `smsSessionSync` |
| GET | `/phone/users/{userId}/sms/sessions` | Get user's SMS sessions | `userSmsSession` |
| GET | `/phone/users/{userId}/sms/sessions/sync` | List user's SMS sessions in descending order | `GetSmsSessions` |

### SMS Campaign

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/sms_campaigns` | List SMS campaigns | `listAccountSMSCampaigns` |
| GET | `/phone/sms_campaigns/{smsCampaignId}` | Get an SMS campaign | `GetSMSCampaign` |
| POST | `/phone/sms_campaigns/{smsCampaignId}/phone_numbers` | Assign a phone number to SMS campaign | `assignCampaignPhoneNumbers` |
| GET | `/phone/sms_campaigns/{smsCampaignId}/phone_numbers/opt_status` | List opt statuses of phone numbers assigned to SMS campaign | `getNumberCampaignOptStatus` |
| PATCH | `/phone/sms_campaigns/{smsCampaignId}/phone_numbers/opt_status` | Update opt statuses of phone numbers assigned to SMS campaign | `updateNumberCampaignOptStatus` |
| DELETE | `/phone/sms_campaigns/{smsCampaignId}/phone_numbers/{phoneNumberId}` | Unassign a phone number | `unassignCampaignPhoneNumber` |
| GET | `/phone/user/{userId}/sms_campaigns/phone_numbers/opt_status` | List user's opt statuses of phone numbers | `getUserNumberCampaignOptStatus` |

### SMS Consent

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/sms_consent/{consentId}/phone_numbers/opt_status` | List opt statuses of phone numbers assigned to SMS consent | `getNumberConsentOptStatus` |

### Users

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/users` | List phone users | `listPhoneUsers` |
| PUT | `/phone/users/batch` | Update multiple users' properties in batch | `updateUsersPropertiesInBatch` |
| POST | `/phone/users/batch` | Batch add users | `batchAddUsers` |
| GET | `/phone/users/{userId}` | Get a user's profile | `phoneUser` |
| PATCH | `/phone/users/{userId}` | Update a user's profile | `updateUserProfile` |
| POST | `/phone/users/{userId}/call_forward_settings/{hourType}` | Add user call forward setting | `addUserCallForwardSetting` |
| DELETE | `/phone/users/{userId}/call_forward_settings/{hourType}` | Delete user call forward setting | `deleteUserCallForwardSetting` |
| GET | `/phone/users/{userId}/call_handling/settings` | Get user call handling setting | `getUserCallHandlingSetting` |
| PATCH | `/phone/users/{userId}/call_handling/settings/{hourType}` | Update user call handling setting | `updateUserCallHandlingSetting` |
| PUT | `/phone/users/{userId}/calling_plans` | Update user's calling plan | `updateCallingPlan` |
| POST | `/phone/users/{userId}/calling_plans` | Assign calling plan to a user | `assignCallingPlan` |
| DELETE | `/phone/users/{userId}/calling_plans/{planType}` | Unassign user's calling plan | `unassignCallingPlan` |
| GET | `/phone/users/{userId}/outbound_caller_id/customized_numbers` | List users' phone numbers for a customized outbound caller ID | `listUserCustomizeOutboundCallerNumbers` |
| POST | `/phone/users/{userId}/outbound_caller_id/customized_numbers` | Add phone numbers for users' customized outbound caller ID | `addUserOutboundCallerNumbers` |
| DELETE | `/phone/users/{userId}/outbound_caller_id/customized_numbers` | Remove users' customized outbound caller ID phone numbers | `deleteUserOutboundCallerNumbers` |
| GET | `/phone/users/{userId}/policies/{policyType}` | Get user policy details | `GetUserPolicyDetails` |
| PATCH | `/phone/users/{userId}/policies/{policyType}` | Update user policy | `updateUserPolicy` |
| POST | `/phone/users/{userId}/profile_settings/{settingType}` | Add a user's profile setting | `addUserProfileSetting` |
| DELETE | `/phone/users/{userId}/profile_settings/{settingType}` | Delete a user's profile profile setting | `deleteUserProfileSetting` |
| GET | `/phone/users/{userId}/settings` | Get a user's profile settings | `phoneUserSettings` |
| PATCH | `/phone/users/{userId}/settings` | Update a user's profile settings | `updateUserSettings` |
| POST | `/phone/users/{userId}/settings/{settingType}` | Add a user's shared access setting | `addUserSetting` |
| DELETE | `/phone/users/{userId}/settings/{settingType}` | Delete a user's shared access setting | `deleteUserSetting` |
| PATCH | `/phone/users/{userId}/settings/{settingType}` | Update a user's shared access setting | `updateUserSetting` |

### Voicemails

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/call_element/{callElementId}/voice_mail` | Get a voicemail by call element ID | `getVoicemailDetailsByCallElementId` |
| GET | `/phone/users/{userId}/call_logs/{id}/voice_mail` | Get user voicemail details from a call log | `getVoicemailDetailsByCallIdOrCallLogId` |
| GET | `/phone/users/{userId}/voice_mails` | Get user's voicemails | `phoneUserVoiceMails` |
| GET | `/phone/voice_mails` | Get account voicemails | `accountVoiceMails` |
| GET | `/phone/voice_mails/download/{fileId}` | Download a phone voicemail | `phoneDownloadVoicemailFile` |
| GET | `/phone/voice_mails/{voicemailId}` | Get voicemail details | `getVoicemailDetails` |
| DELETE | `/phone/voice_mails/{voicemailId}` | Delete a voicemail | `deleteVoicemail` |
| PATCH | `/phone/voice_mails/{voicemailId}` | Update Voicemail Read Status | `updateVoicemailReadStatus` |

### Zoom Rooms

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/phone/rooms` | List Zoom Rooms under Zoom Phone license | `listZoomRooms` |
| POST | `/phone/rooms` | Add a Zoom Room to a Zoom Phone | `addZoomRoom` |
| GET | `/phone/rooms/unassigned` | List Zoom Rooms without Zoom Phone assignment | `listUnassignedZoomRooms` |
| GET | `/phone/rooms/{roomId}` | Get a Zoom Room under Zoom Phone license | `getZoomRoom` |
| DELETE | `/phone/rooms/{roomId}` | Remove a Zoom Room from a ZP account | `RemoveZoomRoom` |
| PATCH | `/phone/rooms/{roomId}` | Update a Zoom Room under Zoom Phone license | `updateZoomRoom` |
| POST | `/phone/rooms/{roomId}/calling_plans` | Assign calling plans to a Zoom Room | `assignCallingPlanToRoom` |
| DELETE | `/phone/rooms/{roomId}/calling_plans/{type}` | Remove a calling plan from a Zoom Room | `unassignCallingPlanFromRoom` |
| POST | `/phone/rooms/{roomId}/phone_numbers` | Assign phone numbers to a Zoom Room | `assignPhoneNumberToZoomRoom` |
| DELETE | `/phone/rooms/{roomId}/phone_numbers/{phoneNumberId}` | Remove a phone number from a Zoom Room | `UnassignPhoneNumberFromZoomRoom` |
