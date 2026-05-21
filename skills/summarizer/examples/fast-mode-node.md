# Fast Mode Node Pattern

Fast mode sends inline transcript text and returns rendered summary text.

```javascript
import { KJUR } from 'jsrsasign';

function generateBuildJwt() {
  const iat = Math.floor(Date.now() / 1000) - 30;
  const exp = iat + 60 * 60;
  return KJUR.jws.JWS.sign(
    'HS256',
    JSON.stringify({ alg: 'HS256', typ: 'JWT' }),
    JSON.stringify({ iss: process.env.ZOOM_API_KEY, iat, exp }),
    process.env.ZOOM_API_SECRET
  );
}

export async function summarizeTranscript(text) {
  const response = await fetch('https://api.zoom.us/v2/aiservices/summarizer/summarize', {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${generateBuildJwt()}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      input: { text },
      config: {
        summary_type: 'conversation',
        task: 'full_summary',
        language: 'en-US',
      },
      reference_id: 'meeting-789',
    }),
  });

  if (!response.ok) throw new Error(await response.text());
  return response.json();
}
```

Expected response shape:

```json
{
  "request_id": "req_123",
  "task": "full_summary",
  "result": {
    "text": "Recap:\\n...\\n\\nSummary:\\n...\\n\\nAction Items:\\n..."
  },
  "model": "zoom-summarizer-v1"
}
```

Do not include `output_format`; current Summarizer docs and quickstart source omit it.
