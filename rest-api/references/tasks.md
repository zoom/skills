# Zoom Tasks API

Task management integrated with Zoom Workplace.

## Overview

Zoom Tasks enables users to create, manage, and track tasks within the Zoom platform. Tasks can be created from meetings, chats, and other Zoom applications, providing unified task management.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with tasks scopes:
- `tasks:read` - Read tasks
- `tasks:write` - Create/manage tasks
- `tasks:read:admin` - Admin read access
- `tasks:write:admin` - Admin write access

## Key Endpoints

### Tasks

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/tasks` | List user tasks |
| POST | `/tasks` | Create task |
| GET | `/tasks/{taskId}` | Get task details |
| PATCH | `/tasks/{taskId}` | Update task |
| DELETE | `/tasks/{taskId}` | Delete task |
| POST | `/tasks/{taskId}/complete` | Mark task complete |

### Task Lists

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/tasks/lists` | List task lists |
| POST | `/tasks/lists` | Create task list |
| GET | `/tasks/lists/{listId}` | Get list details |
| PATCH | `/tasks/lists/{listId}` | Update list |
| DELETE | `/tasks/lists/{listId}` | Delete list |
| GET | `/tasks/lists/{listId}/tasks` | Get tasks in list |

### Assignments

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/tasks/{taskId}/assign` | Assign task to user |
| DELETE | `/tasks/{taskId}/assignees/{userId}` | Unassign user |
| GET | `/tasks/assigned` | Get tasks assigned to me |
| GET | `/tasks/created` | Get tasks created by me |

### Reminders

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/tasks/{taskId}/reminders` | List reminders |
| POST | `/tasks/{taskId}/reminders` | Add reminder |
| DELETE | `/tasks/{taskId}/reminders/{reminderId}` | Remove reminder |

### Comments

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/tasks/{taskId}/comments` | List comments |
| POST | `/tasks/{taskId}/comments` | Add comment |
| DELETE | `/tasks/{taskId}/comments/{commentId}` | Delete comment |

## Example: Create a Task

```bash
curl -X POST "https://api.zoom.us/v2/tasks" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Review Q1 report",
    "description": "Review and provide feedback on quarterly report",
    "due_date": "2024-01-20T17:00:00Z",
    "priority": "high",
    "list_id": "list_abc",
    "assignees": ["user@example.com"]
  }'
```

### Response

```json
{
  "id": "task_xyz123",
  "title": "Review Q1 report",
  "description": "Review and provide feedback on quarterly report",
  "status": "pending",
  "priority": "high",
  "due_date": "2024-01-20T17:00:00Z",
  "created_at": "2024-01-15T10:00:00Z",
  "created_by": "creator@example.com",
  "assignees": [
    {
      "user_id": "user_abc",
      "email": "user@example.com",
      "name": "John Doe"
    }
  ],
  "list": {
    "id": "list_abc",
    "name": "Q1 Tasks"
  }
}
```

## Example: List Tasks

```bash
curl -X GET "https://api.zoom.us/v2/tasks?status=pending&due_before=2024-01-31&page_size=50" \
  -H "Authorization: Bearer {accessToken}"
```

### Response

```json
{
  "page_size": 50,
  "next_page_token": "token123",
  "tasks": [
    {
      "id": "task_xyz123",
      "title": "Review Q1 report",
      "status": "pending",
      "priority": "high",
      "due_date": "2024-01-20T17:00:00Z"
    }
  ]
}
```

## Example: Update Task Status

```bash
curl -X PATCH "https://api.zoom.us/v2/tasks/{taskId}" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "in_progress"
  }'
```

## Task Status

| Status | Description |
|--------|-------------|
| `pending` | Not started |
| `in_progress` | Currently being worked on |
| `completed` | Task finished |
| `cancelled` | Task cancelled |

## Priority Levels

| Priority | Description |
|----------|-------------|
| `low` | Low priority |
| `medium` | Normal priority |
| `high` | High priority |
| `urgent` | Urgent priority |

## Task Sources

Tasks can be created from:
- Zoom Meetings (action items)
- Team Chat messages
- Zoom Docs
- API/Direct creation

## Filters

| Filter | Description |
|--------|-------------|
| `status` | Filter by task status |
| `priority` | Filter by priority |
| `due_before` | Tasks due before date |
| `due_after` | Tasks due after date |
| `list_id` | Tasks in specific list |
| `assignee` | Tasks assigned to user |

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Tasks
- **Zoom Tasks Documentation**: https://developers.zoom.us/docs/zoom-tasks/
