# AI Companion REST API

Access AI-generated meeting summaries, transcripts, and conversation archives.

## Overview

The REST API provides access to AI Companion content:
- Meeting summaries
- Meeting transcripts
- Conversation archives
- AI-related webhooks

## Meeting Summary API

### Get Meeting Summary

```
GET /v2/meetings/{meetingUUID}/meeting_summary
```

**Prerequisites**:
- Meeting Summary feature enabled in account
- Server-to-Server OAuth with admin scopes
- Meeting has ended

**UUID Encoding**: If the meeting UUID starts with `/` or contains `//`, you must double-encode it:

```javascript
function encodeMeetingUUID(uuid) {
  if (uuid.startsWith('/') || uuid.includes('//')) {
    return encodeURIComponent(encodeURIComponent(uuid));
  }
  return encodeURIComponent(uuid);
}
```

### Example

```javascript
async function getMeetingSummary(meetingUUID) {
  const encodedUUID = encodeMeetingUUID(meetingUUID);
  
  const response = await fetch(
    `https://api.zoom.us/v2/meetings/${encodedUUID}/meeting_summary`,
    {
      headers: { 'Authorization': `Bearer ${accessToken}` }
    }
  );
  
  if (!response.ok) {
    throw new Error(`Failed to get summary: ${response.status}`);
  }
  
  return response.json();
}
```

### Response

```json
{
  "meeting_uuid": "abc123def456...",
  "meeting_id": 12345678901,
  "meeting_host_id": "user_xyz",
  "meeting_host_email": "host@example.com",
  "meeting_topic": "Weekly Team Sync",
  "meeting_start_time": "2024-01-15T10:00:00Z",
  "meeting_end_time": "2024-01-15T10:45:00Z",
  "summary_start_time": "2024-01-15T10:00:00Z",
  "summary_end_time": "2024-01-15T10:45:00Z",
  "summary_created_time": "2024-01-15T10:50:00Z",
  "summary_content": {
    "summary": "The team discussed Q1 roadmap priorities and assigned action items...",
    "next_steps": [
      "John to finalize design specs by Friday",
      "Sarah to schedule customer interviews next week",
      "Team to review updated timeline in next meeting"
    ],
    "keywords": ["roadmap", "Q1", "design", "customers", "timeline"]
  }
}
```

### Required Scopes

| Scope | Access Level |
|-------|--------------|
| `meeting:read:admin` | Account-wide access |
| `meeting:read` | User's own meetings |

### Common Errors

| Status | Error | Solution |
|--------|-------|----------|
| 404 | Meeting not found | Check UUID, ensure meeting exists |
| 403 | Forbidden | Make app admin-managed, check scopes |
| 400 | Summary not available | Feature not enabled or summary not generated yet |

---

## Meeting Transcript API

Transcripts are accessed via Cloud Recording endpoints.

### Get Recording with Transcript

```
GET /v2/meetings/{meetingId}/recordings
```

### Example

```javascript
async function getMeetingTranscript(meetingId) {
  const response = await fetch(
    `https://api.zoom.us/v2/meetings/${meetingId}/recordings`,
    {
      headers: { 'Authorization': `Bearer ${accessToken}` }
    }
  );
  
  const data = await response.json();
  
  // Filter for transcript files
  const transcripts = data.recording_files.filter(file => 
    ['TRANSCRIPT', 'CC', 'SUMMARY'].includes(file.file_type)
  );
  
  return transcripts;
}

// Download a transcript file
async function downloadTranscript(downloadUrl) {
  const response = await fetch(downloadUrl, {
    headers: { 'Authorization': `Bearer ${accessToken}` }
  });
  
  return response.text();  // or .json() for JSON format
}
```

### Transcript File Types

| file_type | file_extension | Description |
|-----------|----------------|-------------|
| `TRANSCRIPT` | VTT | WebVTT format transcript |
| `TRANSCRIPT` | JSON | JSON format with timestamps |
| `CC` | VTT | Closed captions |
| `SUMMARY` | JSON | AI-generated summary file |

### VTT Format Example

```
WEBVTT

00:00:05.000 --> 00:00:08.500
John Smith: Good morning everyone, let's get started.

00:00:09.000 --> 00:00:15.000
Sarah Jones: Thanks John. I wanted to discuss the Q1 roadmap.
```

### JSON Format Example

```json
{
  "transcript": [
    {
      "start_time": 5000,
      "end_time": 8500,
      "text": "Good morning everyone, let's get started.",
      "speaker_name": "John Smith",
      "speaker_id": "user_abc"
    },
    {
      "start_time": 9000,
      "end_time": 15000,
      "text": "Thanks John. I wanted to discuss the Q1 roadmap.",
      "speaker_name": "Sarah Jones",
      "speaker_id": "user_xyz"
    }
  ]
}
```

---

## Conversation Archive API

Archive AI Companion panel conversations (available since September 2025).

### Get Conversation Archives

```
GET /v2/users/{userId}/conversation_archive
```

### Example

```javascript
async function getConversationArchives(userId, from, to) {
  const params = new URLSearchParams({
    from: from,  // ISO 8601 date
    to: to
  });
  
  const response = await fetch(
    `https://api.zoom.us/v2/users/${userId}/conversation_archive?${params}`,
    {
      headers: { 'Authorization': `Bearer ${accessToken}` }
    }
  );
  
  return response.json();
}
```

---

## Webhooks

### recording.transcript_completed

Fired when transcript processing is complete.

```json
{
  "event": "recording.transcript_completed",
  "payload": {
    "account_id": "abc123",
    "object": {
      "uuid": "meeting_uuid_here",
      "id": 12345678901,
      "host_id": "user_xyz",
      "topic": "Weekly Sync",
      "start_time": "2024-01-15T10:00:00Z",
      "duration": 45,
      "recording_files": [
        {
          "id": "file_123",
          "file_type": "TRANSCRIPT",
          "file_extension": "VTT",
          "download_url": "https://zoom.us/rec/download/..."
        }
      ]
    }
  }
}
```

### recording.completed

Fired when recording processing is complete (may include summary).

```json
{
  "event": "recording.completed",
  "payload": {
    "account_id": "abc123",
    "object": {
      "uuid": "meeting_uuid_here",
      "recording_files": [
        {
          "file_type": "MP4",
          "download_url": "..."
        },
        {
          "file_type": "SUMMARY",
          "download_url": "..."
        }
      ]
    }
  }
}
```

### Webhook Handler Example

```javascript
app.post('/webhook', async (req, res) => {
  const { event, payload } = req.body;
  
  switch (event) {
    case 'recording.transcript_completed':
      await processTranscript(payload.object);
      break;
      
    case 'recording.completed':
      // Check if summary file is included
      const summaryFile = payload.object.recording_files.find(
        f => f.file_type === 'SUMMARY'
      );
      if (summaryFile) {
        await processSummary(summaryFile);
      }
      break;
  }
  
  res.sendStatus(200);
});
```

---

## Complete Example: AI Content Pipeline

```javascript
class ZoomAIContentPipeline {
  constructor(accessToken) {
    this.accessToken = accessToken;
  }
  
  async getMeetingAIContent(meetingId, meetingUUID) {
    const results = {
      summary: null,
      transcript: null,
      recordings: null
    };
    
    // Get summary
    try {
      results.summary = await this.getMeetingSummary(meetingUUID);
    } catch (e) {
      console.log('Summary not available:', e.message);
    }
    
    // Get recordings with transcripts
    try {
      const recordings = await this.getRecordings(meetingId);
      results.recordings = recordings;
      
      // Extract transcript files
      results.transcript = recordings.recording_files.filter(
        f => f.file_type === 'TRANSCRIPT'
      );
    } catch (e) {
      console.log('Recordings not available:', e.message);
    }
    
    return results;
  }
  
  async getMeetingSummary(uuid) {
    const encodedUUID = this.encodeUUID(uuid);
    const response = await fetch(
      `https://api.zoom.us/v2/meetings/${encodedUUID}/meeting_summary`,
      { headers: { 'Authorization': `Bearer ${this.accessToken}` } }
    );
    
    if (!response.ok) throw new Error(`Status: ${response.status}`);
    return response.json();
  }
  
  async getRecordings(meetingId) {
    const response = await fetch(
      `https://api.zoom.us/v2/meetings/${meetingId}/recordings`,
      { headers: { 'Authorization': `Bearer ${this.accessToken}` } }
    );
    
    if (!response.ok) throw new Error(`Status: ${response.status}`);
    return response.json();
  }
  
  encodeUUID(uuid) {
    if (uuid.startsWith('/') || uuid.includes('//')) {
      return encodeURIComponent(encodeURIComponent(uuid));
    }
    return encodeURIComponent(uuid);
  }
}

// Usage
const pipeline = new ZoomAIContentPipeline(accessToken);
const content = await pipeline.getMeetingAIContent(meetingId, meetingUUID);
```

---

## Required Scopes

| Scope | Description |
|-------|-------------|
| `meeting:read:admin` | Read meeting summaries (account-wide) |
| `meeting:read` | Read own meeting summaries |
| `recording:read:admin` | Read recordings/transcripts (account-wide) |
| `recording:read` | Read own recordings/transcripts |
| `user:read:admin` | Read conversation archives |

---

## Limitations

| Limitation | Notes |
|------------|-------|
| Processing delay | Summaries/transcripts not instant after meeting |
| Admin access | Some endpoints require admin role |
| Feature enablement | AI features must be enabled in account settings |
| No AI panel API | Cannot access AI Companion panel Q&A (except archives) |

---

## Resources

- **Meeting Summary API**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#operation/meetingSummary
- **Cloud Recording API**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/methods/#tag/Cloud-Recording
- **Webhook Events**: https://developers.zoom.us/docs/api/rest/reference/zoom-api/events/
