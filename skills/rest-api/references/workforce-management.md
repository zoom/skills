# Zoom Workforce Management API

Authoritative endpoint inventory for Workforce Management. This file mirrors the official Zoom API Hub OpenAPI document for this product area.

## Canonical Source

- OpenAPI JSON: https://developers.zoom.us/api-hub/workforce-management/methods/endpoints.json
- Base URL: `https://api.zoom.us/v2`
- Authentication details: [authentication.md](authentication.md)

## Notes

- Endpoint methods and paths below are generated from the official Zoom API Hub `paths` object.
- Scope names are defined per operation and frequently use granular scope names. Check the API Hub operation page for the exact scopes before implementation.
- Use this file for endpoint discovery and inventory. Use `../examples/` for orchestration patterns, not as the canonical source of path names.

## Coverage

| Metric | Value |
|--------|-------|
| Endpoint operations | 17 |
| Path templates | 13 |
| Tags | 8 |

## Tag Index

| Tag | Operations |
|-----|------------|
| Filter groups | 1 |
| Forecasts | 2 |
| Imports | 4 |
| Organizational Groups | 5 |
| Reports | 2 |
| Schedules | 1 |
| Scheduling groups | 1 |
| Users | 1 |

## Endpoints by Tag

### Filter groups

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/workforce-management/filter-groups` | List filter groups | `Listfiltergroups` |

### Forecasts

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/workforce-management/forecasts` | List forecasts | `Listforecasts` |
| GET | `/workforce-management/forecasts/{forecastId}/scheduling-groups/{schedulingGroupId}` | Get a forecast for a scheduling group | `Getforecastofschedulinggroup` |

### Imports

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| POST | `/workforce-management/imports/historical-agent-status` | Upload historical agent status | `uploadHistoricalAgentStatus` |
| DELETE | `/workforce-management/imports/historical-agent-status` | Delete historical agent status | `deleteHistoricalAgentStatus` |
| POST | `/workforce-management/imports/historical-queue-metrics` | Upload historical queue metrics | `Uploadhistoricalqueuemetrics` |
| GET | `/workforce-management/imports/{importId}/historical-queue-metrics` | Get historical queue metrics import metadata | `Viewhistoricalqueuemetricimoprtmeta` |

### Organizational Groups

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/workforce-management/organizational-groups` | Get multiple organizational groups | `getMultipleOrganizationalGroups` |
| POST | `/workforce-management/organizational-groups` | Create an organizational group | `createOrganizationalGroup` |
| GET | `/workforce-management/organizational-groups/{organizationalGroupId}` | Get a single organizational group | `getOrganizationalGroup` |
| DELETE | `/workforce-management/organizational-groups/{organizationalGroupId}` | Delete organizational group | `deleteOrganizationalGroup` |
| PATCH | `/workforce-management/organizational-groups/{organizationalGroupId}` | Update an organizational group | `updateOrganizationalGroup` |

### Reports

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/workforce-management/reports/adherence/agents` | List agents' adherence data | `listAdherenceAgents` |
| GET | `/workforce-management/reports/schedules/agents` | List all schedule agents | `listScheduleAgents` |

### Schedules

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/workforce-management/schedules/agents` | List all schedule agents | `listSchedules` |

### Scheduling groups

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/workforce-management/scheduling-groups` | List scheduling groups | `Listschedulinggroups` |

### Users

| Method | Endpoint | Summary | Operation ID |
|--------|----------|---------|-------------|
| GET | `/workforce-management/users` | List users | `Listallusers` |
