# Zoom Mail API

Email service API integrated with Zoom Workplace.

## Overview

Zoom Mail provides email services integrated into the Zoom platform. The API allows programmatic access to email functionality including reading, sending, and managing email messages and folders.

## Base URL

```
https://api.zoom.us/v2
```

## Authentication

Requires OAuth 2.0 with mail scopes:
- `mail:read` - Read email messages
- `mail:write` - Send and manage emails
- `mail:read:admin` - Admin read access
- `mail:write:admin` - Admin write access

## Key Endpoints

### Messages

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/mail/mailboxes/{mailboxId}/messages` | List messages |
| GET | `/mail/mailboxes/{mailboxId}/messages/{messageId}` | Get message details |
| POST | `/mail/mailboxes/{mailboxId}/messages` | Send/create message |
| DELETE | `/mail/mailboxes/{mailboxId}/messages/{messageId}` | Delete message |
| PATCH | `/mail/mailboxes/{mailboxId}/messages/{messageId}` | Update message (read/unread, move) |

### Folders

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/mail/mailboxes/{mailboxId}/folders` | List folders |
| POST | `/mail/mailboxes/{mailboxId}/folders` | Create folder |
| GET | `/mail/mailboxes/{mailboxId}/folders/{folderId}` | Get folder details |
| PATCH | `/mail/mailboxes/{mailboxId}/folders/{folderId}` | Update folder |
| DELETE | `/mail/mailboxes/{mailboxId}/folders/{folderId}` | Delete folder |

### Mailboxes

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/mail/mailboxes` | List user mailboxes |
| GET | `/mail/mailboxes/{mailboxId}` | Get mailbox details |
| GET | `/mail/mailboxes/{mailboxId}/settings` | Get mailbox settings |
| PATCH | `/mail/mailboxes/{mailboxId}/settings` | Update mailbox settings |

### Attachments

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/mail/mailboxes/{mailboxId}/messages/{messageId}/attachments` | List attachments |
| GET | `/mail/mailboxes/{mailboxId}/messages/{messageId}/attachments/{attachmentId}` | Download attachment |

### Drafts

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/mail/mailboxes/{mailboxId}/drafts` | List drafts |
| POST | `/mail/mailboxes/{mailboxId}/drafts` | Create draft |
| PATCH | `/mail/mailboxes/{mailboxId}/drafts/{draftId}` | Update draft |
| DELETE | `/mail/mailboxes/{mailboxId}/drafts/{draftId}` | Delete draft |
| POST | `/mail/mailboxes/{mailboxId}/drafts/{draftId}/send` | Send draft |

## Example: Send Email

```bash
curl -X POST "https://api.zoom.us/v2/mail/mailboxes/{mailboxId}/messages" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "to": [{"email": "recipient@example.com", "name": "John Doe"}],
    "subject": "Meeting Follow-up",
    "body": {
      "content_type": "text/html",
      "content": "<p>Thank you for joining the meeting today.</p>"
    },
    "cc": [],
    "bcc": []
  }'
```

### Response

```json
{
  "id": "msg_abc123",
  "thread_id": "thread_xyz",
  "from": {
    "email": "sender@example.com",
    "name": "Sender Name"
  },
  "to": [{"email": "recipient@example.com", "name": "John Doe"}],
  "subject": "Meeting Follow-up",
  "sent_time": "2024-01-15T10:30:00Z",
  "status": "sent"
}
```

## Example: List Messages

```bash
curl -X GET "https://api.zoom.us/v2/mail/mailboxes/{mailboxId}/messages?folder_id=inbox&page_size=50" \
  -H "Authorization: Bearer {accessToken}"
```

## Standard Folders

| Folder | Description |
|--------|-------------|
| `inbox` | Incoming messages |
| `sent` | Sent messages |
| `drafts` | Draft messages |
| `trash` | Deleted messages |
| `spam` | Spam/junk messages |
| `archive` | Archived messages |

## Message Flags

| Flag | Description |
|------|-------------|
| `read` | Message has been read |
| `starred` | Message is starred/flagged |
| `important` | Message marked as important |

## Resources

- **API Reference**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Mail
- **Zoom Mail Documentation**: https://developers.zoom.us/docs/zoom-mail/
