<!-- Source: https://developers.zoom.us/docs/video-sdk/web/recording.md -->
<!-- Fetched: 2026-06-15 -->

# Cloud recording


Zoom Video SDK sessions offer cloud recording using:

-   [In-session SDK functions](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.RecordingClient.html) to enable session hosts to start, pause, resume, and stop recordings. See the **SDK** tab for these functions.
-   [Cloud recording APIs](/docs/api/video-sdk/#tag/cloud-recording) to enable Developers to initiate and manage cloud recording functions. See the **REST API** tab for details.

_Recording resolution depends on the video quality being sent from the Video SDK, with a maximum resolution of 1080p on web._

**Zoom recommends that you provide an in-product notice to your end-users when a user enables cloud recording.**

## Prerequisites

-   [Video SDK Account](https://zoom.us/pricing/developer)
-   [Cloud Recording Storage Plan](https://zoom.us/billing)

## Recording options

You can record Video SDK sessions in many different ways, such as:

**Video or screen sharing:**

-   Record active speaker with shared screen
-   Record gallery view with shared screen
-   Record active speaker, gallery view and shared screen separately

**Audio:**

-   Record audio-only files
-   Record one audio file for all users
-   Record a separate audio file of each user

**Chat:**

-   Save chat messages from the session

**Advanced:**

-   Add a timestamp to the recording
-   Display users' names in the recording
-   Record thumbnails when sharing
-   Optimize the recording for a third party video editor

**Record user video files**

You can also record individual user video files. To do so, [contact Developer Support](/support/) and ask for the ability to "Enable ISO Video for V-SDK".

Once enabled, the host must set the `cloud_recording_option` property to `1` in the [Video SDK JWT](/docs/video-sdk/auth/#payload). Users must also set the `cloud_recording_election` property to `1` in the their [Video SDK JWT](/docs/video-sdk/auth/#payload) to allow individual recording.

Recordings do not include transcription and translations, but you can separately add these features to your integration. See the **Live transcription and translation** documentation for details.

## Init cloud recording

After joining a session, call `client.getRecordingClient()` to get the cloud recording client.

```javascript
const cloudRecording = client.getRecordingClient();
```

## Start recording

```javascript
cloudRecording.startCloudRecording();
```

To start recording, PATCH `recording.start` to the `/events` endpoint.

Endpoint: [`https://api.zoom.us/v2/videosdk/sessions/{sessionId}/events`](/docs/api/video-sdk/#tag/sessions/PATCH/videosdk/sessions/{sessionId}/events)

**Request header:**

```json
{
    "Authorization": "Bearer {VIDEOSDK_JWT_TOKEN}"
}
```

**Request body:**

```json
{
    "method": "recording.start"
}
```

**Response code:**

`202 Accepted`

**Response body:**

_Empty_

## Stop recording

```javascript
cloudRecording.stopCloudRecording();
```

To stop recording, PATCH `recording.stop` to the `/events` endpoint.

Endpoint: [`https://api.zoom.us/v2/videosdk/sessions/{sessionId}/events`](/docs/api/video-sdk/#tag/sessions/PATCH/videosdk/sessions/{sessionId}/events)

**Request header:**

```json
{
    "Authorization": "Bearer {VIDEOSDK_JWT_TOKEN}"
}
```

    **Request body:**

```json
{
    "method": "recording.stop"
}
```

**Response code:**

`202 Accepted`

**Response body:**

_Empty_

## Spotlight user

Cloud recording shows the active speaker when you record video to the cloud with the **Cloud recording setting**:

-   **Record active speaker with shared screen** or
-   **Record active speaker, gallery view and shared screen separately**, with **Active speaker** selected.

See [Account: Cloud recording](/docs/video-sdk/account/#cloud-recording) for details.

This may cause the active speaker to change unexpectedly, for example if another user is unmuted and coughs. If you want to prevent this, or keep one speaker as the active speaker in the cloud recording, you can spotlight a user.

A host or manager can use [`spotlightVideo`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#spotlightVideo) to spotlight a user's video in the cloud recording. This returns the user ID for the spotlit user in the [`video-active-change`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.html#event_video_active_change) event. The spotlit user remains the active user, regardless of whether they are speaking. This also impacts the behavior in the recording file. When you select a record active speaker option, the recording will always show the spotlit user's video.

Use [`removeSpotlightedVideo`](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.Stream.html#removeSpotlightedVideo) to remove the user's video from being spotlit.

## Get account recordings

_REST API only._

To get all of the cloud recordings for the account, GET `/recordings`.

Endpoint: [`https://api.zoom.us/v2/videosdk/recordings`](/docs/api/video-sdk/#tag/cloud-recording/GET/videosdk/recordings)

## Get session recordings

_REST API only._

To get all the recordings for the session, GET `{sessionId}/recordings`.

Endpoint: [`https://api.zoom.us/v2/videosdk/sessions/{sessionId}/recordings`](/docs/api/video-sdk/#tag/cloud-recording/GET/videosdk/sessions/{sessionId}/recordings)

## Recording completed webhook

You can also receive a POST request to your server with JSON (a webhook) once the recording has completed processing. You can use Webhooks to automate distribution or uploading of the recording files to your own cloud storage system. See [Using Zoom Webhooks](/docs/api/webhooks/) and the [`session.recording_completed`](/docs/api/video-sdk/events/#tag/session/POSTsession.recording_completed) webhook for details.

For more details, see the full [Video SDK API](/docs/api/video-sdk) and [Webhook](/docs/api/video-sdk/events/) reference pages.

## View recordings

Recordings are also available to view and download from your [Video SDK Account Recording page](https://zoom.us/recording).

## Get transcript and summary

To add a transcript or AI summary to the cloud recording, set the `cloud_recording_transcript_option` field in the JWT used to start the session.

-   `0` (default) - Neither transcript nor summary
-   `1` - Transcript only
-   `2` - Transcript and summary

See the final setting in the [JWT payload table](/docs/video-sdk/auth/#payload) for details.

> **Get AI summary**
>
> To receive the AI summary, enable both cloud recording (in the SDK) and session transcript (`cloud_recording_transcript_option` JWT field set to `2`). Each feature incurs a separate charge. See the [Zoom Developer pricing page](https://zoom.us/pricing/developer) for details.

### Start cloud recording

See the platform get started documentation for details about how to start and join a session. After joining a session, call `startCloudRecording` to start recording. This will also start the transcript and AI summary autogeneration if you enabled it in the JWT.

### Transcript and summary webhook notifications

After a session ends, it may take some time to generate the transcript and AI summary. You can use webhooks to receive notifications when they're ready.
Subscribe to the Video SDK [`session.recording_transcript_completed`](/docs/api/video-sdk/events/#tag/session/postsession.recording_transcript_completed) webhook to be notified when the transcript is completed, and [`session.recording_summary_completed`](/docs/api/video-sdk/events/#tag/session/postsession.recording_summary_completed) to be notified when the summary is completed. See [Using webhooks](/docs/api/webhooks/) for setup details.

_**Note: These webhooks are available to Zoom Build Platform universal credit accounts.**_

### Get download links

Once the transcript or AI summary is generated, you can retrieve it by calling:

```bash
GET /videosdk/sessions/{sessionId}/recordings
```

Sample response:

```json
{
    "timezone": "",
    "duration": 1,
    "session_id": "oiFKKKL/RyGdm9W1r1Ll6w==",
    "session_name": "willie3",
    "session_key": "",
    "start_time": "2025-05-09T11:02:09Z",
    "total_size": 4869060,
    "recording_count": 3,
    "recording_files": [
        {
            "id": "aaa111",
            "status": "completed",
            "recording_start": "2025-05-09T11:02:22Z",
            "recording_end": "2025-05-09T11:03:24Z",
            "file_type": "MP4",
            "file_size": 3892799,
            "download_url": "https://example.com/download_url12345",
            "recording_type": "shared_screen_with_speaker_view",
            "file_extension": "MP4"
        },
        {
            "id": "bbb222",
            "status": "completed",
            "recording_start": "2025-05-09T11:02:22Z",
            "recording_end": "2025-05-09T11:03:24Z",
            "file_type": "M4A",
            "file_size": 974973,
            "download_url": "https://example.com/download_url55555",
            "recording_type": "audio_only",
            "file_extension": "M4A"
        },
        {
            "id": "ccc333",
            "status": "completed",
            "recording_start": "2025-05-09T11:02:22Z",
            "recording_end": "2025-05-09T11:03:24Z",
            "file_type": "TIMELINE",
            "file_size": 22543,
            "download_url": "https://example.com/download_url67890",
            "recording_type": "timeline",
            "file_extension": "JSON"
        },
        {
            "id": "ddd444",
            "status": "completed",
            "recording_start": "2025-05-09T11:02:22Z",
            "recording_end": "2025-05-09T11:03:24Z",
            "file_type": "TRANSCRIPT",
            "file_size": 1288,
            "download_url": "https://example.com/download_url09876",
            "recording_type": "audio_transcript",
            "file_extension": "VTT"
        },
        {
            "id": "eee555",
            "status": "completed",
            "recording_start": "2025-05-09T11:02:22Z",
            "recording_end": "2025-05-09T11:03:24Z",
            "file_type": "SUMMARY",
            "file_size": 1039,
            "download_url": "https://example.com/download_url54321",
            "recording_type": "summary",
            "file_extension": "JSON"
        }
    ],
    "download_access_token": "XXXXXXX"
}
```

### Download

Use the `download_url` for files with `file_type` of `TRANSCRIPT` or `SUMMARY` to download the transcript or AI summary.

-   `TRANSCRIPT` - a VTT file containing the time-coded session transcript. The response includes this file when you enable the session transcript (`cloud_recording_transcript_option` set to `1` or `2`).
-   `SUMMARY` - a JSON file containing the AI-generated summary of the session. The response includes this file only when you enable both cloud recording and session transcript (`cloud_recording_transcript_option` set to `2`), and your account is billed for both features. The current summary output is limited in scope and only includes the `summary` section.

## More cloud recording features

For the full set of cloud recording features, see [RecordingClient in the Video SDK Reference](https://marketplacefront.zoom.us/sdk/custom/web/modules/ZoomVideo.RecordingClient.html).

## From the developer blog

See the following blog post for a sample walkthrough.

-   [How to play a Zoom Video SDK cloud recording](/blog/how-to-play-video-sdk-recording) by Will Ezrine - 03-21-2024
