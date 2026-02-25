# SCIM 2.0 API

System for Cross-domain Identity Management for automated user provisioning.

## Overview

Zoom's SCIM 2.0 API enables identity providers (IdPs) to automatically provision, update, and deprovision users. This standard protocol integrates with enterprise identity management systems like Okta, Azure AD, OneLogin, and others.

## Base URL

```
https://api.zoom.us/scim2
```

## Authentication

SCIM endpoints use Bearer token authentication:

```
Authorization: Bearer {scim_token}
```

Generate SCIM token in: Zoom Admin Portal → Advanced → SCIM

## Key Endpoints

### Users

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/Users` | List users |
| POST | `/Users` | Create user |
| GET | `/Users/{id}` | Get user |
| PUT | `/Users/{id}` | Replace user |
| PATCH | `/Users/{id}` | Update user |
| DELETE | `/Users/{id}` | Delete user |

### Groups

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/Groups` | List groups |
| POST | `/Groups` | Create group |
| GET | `/Groups/{id}` | Get group |
| PUT | `/Groups/{id}` | Replace group |
| PATCH | `/Groups/{id}` | Update group |
| DELETE | `/Groups/{id}` | Delete group |

### Service Provider Config

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/ServiceProviderConfig` | Get service provider capabilities |
| GET | `/ResourceTypes` | Get resource types |
| GET | `/Schemas` | Get schemas |

## Example: Create User

```bash
curl -X POST "https://api.zoom.us/scim2/Users" \
  -H "Authorization: Bearer {scim_token}" \
  -H "Content-Type: application/scim+json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "userName": "john.doe@company.com",
    "name": {
      "givenName": "John",
      "familyName": "Doe"
    },
    "emails": [
      {
        "value": "john.doe@company.com",
        "primary": true
      }
    ],
    "active": true,
    "userType": "Licensed",
    "urn:ietf:params:scim:schemas:extension:zoom:2.0:User": {
      "department": "Engineering",
      "jobTitle": "Software Engineer",
      "phoneNumbers": [
        {
          "value": "+14155551234",
          "type": "work"
        }
      ]
    }
  }'
```

### Response

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "id": "abc123xyz",
  "userName": "john.doe@company.com",
  "name": {
    "givenName": "John",
    "familyName": "Doe",
    "formatted": "John Doe"
  },
  "emails": [
    {
      "value": "john.doe@company.com",
      "primary": true
    }
  ],
  "active": true,
  "userType": "Licensed",
  "meta": {
    "resourceType": "User",
    "created": "2024-01-25T10:00:00Z",
    "lastModified": "2024-01-25T10:00:00Z",
    "location": "https://api.zoom.us/scim2/Users/abc123xyz"
  }
}
```

## Example: List Users with Filter

```bash
curl "https://api.zoom.us/scim2/Users?filter=userName%20eq%20%22john.doe@company.com%22" \
  -H "Authorization: Bearer {scim_token}"
```

### Common Filters

```
# By email
filter=userName eq "user@company.com"

# By active status
filter=active eq true

# By department
filter=urn:ietf:params:scim:schemas:extension:zoom:2.0:User:department eq "Engineering"

# Combined
filter=active eq true and userType eq "Licensed"
```

## Example: Update User (PATCH)

```bash
curl -X PATCH "https://api.zoom.us/scim2/Users/{id}" \
  -H "Authorization: Bearer {scim_token}" \
  -H "Content-Type: application/scim+json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [
      {
        "op": "replace",
        "path": "active",
        "value": false
      },
      {
        "op": "replace",
        "path": "name.givenName",
        "value": "Jonathan"
      },
      {
        "op": "add",
        "path": "phoneNumbers",
        "value": [
          {
            "value": "+14155559999",
            "type": "mobile"
          }
        ]
      }
    ]
  }'
```

## Example: Create Group

```bash
curl -X POST "https://api.zoom.us/scim2/Groups" \
  -H "Authorization: Bearer {scim_token}" \
  -H "Content-Type: application/scim+json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
    "displayName": "Engineering Team",
    "members": [
      {
        "value": "abc123xyz",
        "display": "john.doe@company.com"
      }
    ]
  }'
```

## User Types

| Type | Description |
|------|-------------|
| `Licensed` | Full Zoom user with license |
| `Basic` | Basic (free) user |
| `None` | No license assigned |

## Zoom SCIM Extension

Zoom-specific attributes use the extension schema:

```
urn:ietf:params:scim:schemas:extension:zoom:2.0:User
```

### Extension Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `department` | String | User's department |
| `jobTitle` | String | Job title |
| `costCenter` | String | Cost center |
| `company` | String | Company name |
| `manager` | Reference | Manager's user ID |
| `phoneNumbers` | Array | Phone numbers |
| `zoomRole` | String | Zoom role ID |

## Pagination

```bash
# List with pagination
curl "https://api.zoom.us/scim2/Users?startIndex=1&count=100" \
  -H "Authorization: Bearer {scim_token}"
```

### Response

```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
  "totalResults": 500,
  "startIndex": 1,
  "itemsPerPage": 100,
  "Resources": [...]
}
```

## Group Membership Operations

### Add Members

```bash
curl -X PATCH "https://api.zoom.us/scim2/Groups/{groupId}" \
  -H "Authorization: Bearer {scim_token}" \
  -H "Content-Type: application/scim+json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [
      {
        "op": "add",
        "path": "members",
        "value": [
          {"value": "user_id_1"},
          {"value": "user_id_2"}
        ]
      }
    ]
  }'
```

### Remove Members

```bash
curl -X PATCH "https://api.zoom.us/scim2/Groups/{groupId}" \
  -H "Authorization: Bearer {scim_token}" \
  -H "Content-Type: application/scim+json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [
      {
        "op": "remove",
        "path": "members[value eq \"user_id_1\"]"
      }
    ]
  }'
```

## Supported IdPs

| Provider | Documentation |
|----------|---------------|
| Okta | https://support.zoom.us/hc/en-us/articles/115005887566 |
| Azure AD | https://support.zoom.us/hc/en-us/articles/360034967951 |
| OneLogin | https://support.zoom.us/hc/en-us/articles/360034831631 |
| Ping Identity | https://support.zoom.us/hc/en-us/articles/360034831471 |

## Error Responses

```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:Error"],
  "status": "400",
  "scimType": "invalidValue",
  "detail": "Invalid email format"
}
```

### Common Error Codes

| Code | Description |
|------|-------------|
| 400 | Invalid request |
| 401 | Authentication failed |
| 403 | Permission denied |
| 404 | Resource not found |
| 409 | Conflict (duplicate) |
| 429 | Rate limited |

## Best Practices

1. **Use PATCH for updates** - More efficient than PUT
2. **Handle pagination** - Always page through large results
3. **Validate before sync** - Test with small user sets first
4. **Monitor sync status** - Check IdP logs for errors
5. **Map attributes correctly** - Ensure IdP→Zoom mapping is accurate

## Resources

- **SCIM Setup Guide**: https://support.zoom.us/hc/en-us/articles/360034967491
- **SCIM 2.0 RFC**: https://datatracker.ietf.org/doc/html/rfc7644
- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/scim-api/methods/
