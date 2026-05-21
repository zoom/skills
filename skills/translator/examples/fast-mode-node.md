# Fast Mode Node Pattern

Fast mode sends one text value and returns a translations map.

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

export async function translateText(text) {
  const response = await fetch('https://api.zoom.us/v2/aiservices/translator/translate', {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${generateBuildJwt()}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      text,
      config: {
        source_language: 'en-US',
        target_languages: ['es-ES'],
      },
      reference_id: 'msg_456',
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
  "result": {
    "translations": {
      "es-ES": "Translated text here."
    }
  }
}
```

Use one target locale per request.
