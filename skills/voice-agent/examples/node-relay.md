# Minimal Node.js relay

Use a server-side relay because browser WebSocket clients cannot set `x-api-key`.

```js
import { createServer } from 'node:http';
import WebSocket, { WebSocketServer } from 'ws';

const ZOOM_URL = 'wss://api.zoom.us/v2/ws/aiservices/voice-agent/live';
const API_KEY = process.env.ZOOM_API_KEY;
if (!API_KEY) throw new Error('ZOOM_API_KEY is required');

const server = createServer();
const relay = new WebSocketServer({ noServer: true, maxPayload: 1024 * 1024 });

server.on('upgrade', (request, socket, head) => {
  if (request.url !== '/live') {
    socket.destroy();
    return;
  }
  relay.handleUpgrade(request, socket, head, (client) => {
    relay.emit('connection', client);
  });
});

relay.on('connection', (client) => {
  const upstream = new WebSocket(ZOOM_URL, ['voice-agent.zoom.v2'], {
    headers: { 'x-api-key': API_KEY },
    maxPayload: 1024 * 1024,
  });

  let ready = false;
  const queued = [];

  upstream.on('open', () => {
    upstream.send(JSON.stringify({
      type: 'session.update',
      session: {
        type: 'realtime',
        listen: {
          transcription: { language: 'en' },
          format: { type: 'pcm16', rate: 16000 },
        },
        think: {
          provider: { type: 'zoom', model: 'gemma-4' },
          instructions: 'Respond naturally and conversationally.',
        },
        speak: { format: { type: 'pcm16' }, voice: 'ariel' },
      },
    }));
  });

  upstream.on('message', (data, isBinary) => {
    if (client.readyState !== WebSocket.OPEN) return;
    if (isBinary) {
      client.send(data, { binary: true });
      return;
    }
    try {
      const event = JSON.parse(data.toString());
      if (event.type === 'session.updated') {
        ready = true;
        for (const frame of queued.splice(0)) upstream.send(frame);
      }
      client.send(JSON.stringify(event));
    } catch {
      client.close();
    }
  });

  client.on('message', (data, isBinary) => {
    if (isBinary) return;
    const text = data.toString();
    if (!ready) {
      queued.push(text);
      return;
    }
    upstream.send(text);
  });

  const close = () => {
    if (upstream.readyState === WebSocket.OPEN) upstream.close();
    if (client.readyState === WebSocket.OPEN) client.close();
  };
  upstream.on('close', close);
  upstream.on('error', close);
  client.on('close', close);
  client.on('error', close);
});

server.listen(Number(process.env.PORT || 4000), '127.0.0.1');
```
