# REST API - Users

User management endpoints for Zoom accounts.

## Overview

Manage Zoom users programmatically - list, create, update, and delete users. This is essential for:
- Automated user provisioning (HR system integration)
- Bulk user management
- User lifecycle management (onboarding/offboarding)
- Skill chaining: Create users before scheduling meetings on their behalf

## Endpoints

### List Users

```bash
GET /users
```

**Query parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | `active`, `inactive`, or `pending` |
| `page_size` | integer | Results per page (default 30, max 300) |
| `next_page_token` | string | Token for pagination |
| `role_id` | string | Filter by role |
| `include_fields` | string | `custom_attributes`, `host_key` |

**Response:**
```json
{
  "page_count": 1,
  "page_number": 1,
  "page_size": 30,
  "total_records": 2,
  "next_page_token": "",
  "users": [
    {
      "id": "abcD3ojkdbjfg",
      "first_name": "John",
      "last_name": "Doe",
      "email": "john@example.com",
      "type": 2,
      "status": "active",
      "created_at": "2024-01-15T10:30:00Z"
    }
  ]
}
```

### Get User

```bash
GET /users/{userId}
```

**Path parameters:**
- `userId` - User ID or email address (or `me` for current user)

**Response includes:** `id`, `first_name`, `last_name`, `email`, `type`, `role_name`, `pmi`, `timezone`, `created_at`, `last_login_time`, `host_key`, etc.

### Create User

```bash
POST /users
```

**IMPORTANT:** The `action` field determines how the user is created.

#### Action Types

| Action | Description | Email Sent? |
|--------|-------------|-------------|
| `create` | Standard creation - sends email, user sets password | Yes |
| `autoCreate` | Auto-provision with temporary password | No |
| `custCreate` | Create without email (managed by SSO) | No |
| `ssoCreate` | SSO-managed user, added to default group | No |

#### Example: Standard Create

```json
{
  "action": "create",
  "user_info": {
    "email": "john.doe@example.com",
    "type": 2,
    "first_name": "John",
    "last_name": "Doe"
  }
}
```

#### Example: Auto-Create (No Email)

```json
{
  "action": "autoCreate",
  "user_info": {
    "email": "jane.doe@example.com",
    "type": 2,
    "first_name": "Jane",
    "last_name": "Doe",
    "password": "TempPass123!"
  }
}
```

#### Example: SSO Create

```json
{
  "action": "ssoCreate",
  "user_info": {
    "email": "sso.user@example.com",
    "type": 2,
    "first_name": "SSO",
    "last_name": "User"
  }
}
```

**Success Response (201):**
```json
{
  "id": "abcD3ojkdbjfg",
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@example.com",
  "type": 2
}
```

### Update User

```bash
PATCH /users/{userId}
```

**Request body (all fields optional):**
```json
{
  "first_name": "Jonathan",
  "last_name": "Doe",
  "type": 2,
  "pmi": 1234567890,
  "timezone": "America/Los_Angeles",
  "language": "en-US",
  "dept": "Engineering",
  "job_title": "Senior Developer",
  "company": "Acme Corp"
}
```

### Delete User

```bash
DELETE /users/{userId}
```

**Query parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `action` | string | `disassociate` (default) or `delete` |
| `transfer_email` | string | Email to transfer meetings/webinars |
| `transfer_meeting` | boolean | Transfer scheduled meetings |
| `transfer_webinar` | boolean | Transfer scheduled webinars |
| `transfer_recording` | boolean | Transfer cloud recordings |

**Example - Delete and transfer:**
```bash
DELETE /users/abcD3ojkdbjfg?action=delete&transfer_email=admin@example.com&transfer_meeting=true
```

## User Types

| Type | Value | Description | License Required |
|------|-------|-------------|------------------|
| Basic | 1 | Free user, limited features | No |
| Licensed | 2 | Full Zoom license | Yes |
| On-prem | 3 | On-premise deployment | Special |
| None | 99 | No plan (deactivated) | No |

## Error Responses

### 409 Conflict - User Exists

```json
{
  "code": 409,
  "message": "User already exists: john@example.com"
}
```

### 400 Bad Request - Validation Error

```json
{
  "code": 300,
  "message": "Validation Failed",
  "errors": [
    {
      "field": "email",
      "message": "Invalid email format"
    }
  ]
}
```

### 404 Not Found

```json
{
  "code": 1001,
  "message": "User does not exist: invalid-user-id"
}
```

## Code Examples

### JavaScript/Node.js - Create User

```javascript
const axios = require('axios');

async function createZoomUser(accessToken, userData) {
  const response = await axios.post(
    'https://api.zoom.us/v2/users',
    {
      action: 'autoCreate',
      user_info: {
        email: userData.email,
        type: 2,
        first_name: userData.firstName,
        last_name: userData.lastName,
        password: generateTempPassword()
      }
    },
    {
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json'
      }
    }
  );
  
  return response.data;
}

// Usage
const newUser = await createZoomUser(token, {
  email: 'new.employee@company.com',
  firstName: 'New',
  lastName: 'Employee'
});
console.log(`Created user: ${newUser.id}`);
```

### JavaScript/Node.js - List All Users (with pagination)

```javascript
async function listAllZoomUsers(accessToken) {
  const users = [];
  let nextPageToken = '';
  
  do {
    const params = new URLSearchParams({
      page_size: '300',
      status: 'active'
    });
    
    if (nextPageToken) {
      params.append('next_page_token', nextPageToken);
    }
    
    const response = await axios.get(
      `https://api.zoom.us/v2/users?${params}`,
      {
        headers: { 'Authorization': `Bearer ${accessToken}` }
      }
    );
    
    users.push(...response.data.users);
    nextPageToken = response.data.next_page_token || '';
    
  } while (nextPageToken);
  
  return users;
}
```

## Skill Chaining

Users API is commonly chained with other skills:

| Chain | Pattern | Use Case |
|-------|---------|----------|
| Users → Meetings | Create user, then schedule meetings | New employee onboarding |
| Users → Webhooks | Create user, subscribe to user events | User lifecycle tracking |
| Users → Recordings | Get user, list their recordings | Compliance/audit |

**Example: Create user then schedule meeting**
```javascript
// Step 1: Create the user
const user = await createZoomUser(token, { email, firstName, lastName });

// Step 2: Schedule meeting for the new user
const meeting = await axios.post(
  `https://api.zoom.us/v2/users/${user.id}/meetings`,
  {
    topic: 'Onboarding Session',
    type: 2,
    start_time: '2024-02-01T10:00:00Z',
    duration: 60
  },
  { headers: { 'Authorization': `Bearer ${token}` }}
);
```

See [user-and-meeting-creation.md](../../general/use-cases/user-and-meeting-creation.md) for complete skill chaining patterns.

## Required Scopes

| Scope | Operations |
|-------|------------|
| `user:read:admin` | List users, get user details (account-wide) |
| `user:write:admin` | Create, update, delete users (account-wide) |
| `user:read:user` | Get own user details |
| `user:write:user` | Update own user details |

## Rate Limits

- **Light**: GET operations (30 requests/second)
- **Medium**: POST/PATCH operations (10 requests/second)  
- **Heavy**: DELETE operations (10 requests/second)

Daily limit: 5000 API calls per app.

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Users
- **User provisioning guide**: https://developers.zoom.us/docs/api/rest/user-provisioning/
- **SCIM 2.0 for enterprise**: See [scim2.md](scim2.md)
