# Environment Variables

## Required for AI Services

| Variable | Required | Where to find it | Use |
|----------|----------|------------------|-----|
| `ZOOM_API_KEY` | Yes | Zoom Build Platform API keys area | JWT issuer (`iss`) |
| `ZOOM_API_SECRET` | Yes | Zoom Build Platform API keys area | HS256 signing secret |

## Required for Batch Jobs Using S3

| Variable | Required | Use |
|----------|----------|-----|
| `AWS_ACCESS_KEY_ID` | When using S3 auth in env | Passed to Zoom for S3 read/write |
| `AWS_SECRET_ACCESS_KEY` | When using S3 auth in env | Passed to Zoom for S3 read/write |
| `AWS_SESSION_TOKEN` | For temporary credentials | Passed to Zoom for short-lived S3 access |
| `WEBHOOK_SECRET` | If using notifications | Verifies batch webhook signatures |

## Local App Variables

| Variable | Required | Use |
|----------|----------|-----|
| `PORT` | No | Local API server port |

Keep Zoom and AWS secrets server-side. Browser clients should call your backend, not Zoom AI Services directly.
