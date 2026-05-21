# Batch Webhook Pipeline

Use batch mode when translating text files stored in S3.

## Submit Job

```bash
curl -X POST https://api.zoom.us/v2/aiservices/translator/jobs \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "mode": "PREFIX",
      "source": "S3",
      "uri": "s3://acme-bucket/input/2026/03/",
      "filters": {
        "include_globs": ["*.txt"],
        "exclude_globs": []
      },
      "auth": {
        "aws": {
          "access_key_id": "AKIA...",
          "secret_access_key": "secret",
          "session_token": "optional"
        }
      }
    },
    "output": {
      "destination": "S3",
      "uri": "s3://acme-bucket/output/2026/03/",
      "layout": "PREFIX",
      "overwrite": false,
      "auth": {
        "aws": {
          "access_key_id": "AKIA...",
          "secret_access_key": "secret",
          "session_token": "optional"
        }
      }
    },
    "config": {
      "source_language": "en-US",
      "target_languages": ["fr-FR"]
    },
    "notifications": {
      "webhook_url": "https://example.com/hooks/translator",
      "secret": "hmac-secret"
    },
    "reference_id": "batch-translation-2026-03"
  }'
```

## Monitor

Poll these endpoints or rely on webhook notifications:

- `GET /aiservices/translator/jobs/{jobId}`
- `GET /aiservices/translator/jobs/{jobId}/files`
- `GET /aiservices/translator/jobs/{jobId}/files/{fileId}`

## Verify Webhooks

When `notifications.secret` is set, verify:

```javascript
import crypto from 'node:crypto';

function verifyZoomAiWebhook({ rawBody, timestamp, signature, secret }) {
  const message = `v0:${timestamp}:${rawBody}`;
  const expected = `sha256=${crypto.createHmac('sha256', secret).update(message).digest('hex')}`;
  return crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expected));
}
```

Use raw request body bytes. Verifying after JSON parsing can change the signed payload.
