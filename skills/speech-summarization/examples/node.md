# Official Node.js sample

This mirrors the current official Speech Summarization sample. It requires Node.js 22 or newer and a server-side API key.

```js
import { readFile } from 'node:fs/promises';
import { basename } from 'node:path';
import { createInterface } from 'node:readline';
import { Readable } from 'node:stream';

const url = 'https://api.zoom.us/v2/aiservices/speech-summarization/summarize';
const apiKey = process.env.ZOOM_API_KEY;
if (!apiKey) throw new Error('ZOOM_API_KEY is required');

const audioPath = process.env.SPEECH_AUDIO_PATH || 'recording.wav';
const config = {
  transcribe: {
    language: 'en-US',
    word_time_offsets: false,
    channel_separation: false,
    diarization: true,
    profanity_filter: false,
    output_format: 'json',
  },
  summary: {
    language: 'en-US',
    citations: true,
    should_remove_sentiment: false,
    pii_redaction: false,
    enable_dynamic_template: true,
  },
};

const form = new FormData();
form.append('config', JSON.stringify(config));
form.append(
  'file',
  new Blob([await readFile(audioPath)], { type: 'application/octet-stream' }),
  basename(audioPath),
);

const response = await fetch(url, {
  method: 'POST',
  headers: { 'x-api-key': apiKey, Accept: 'text/event-stream' },
  body: form,
});
if (!response.ok) throw new Error(await response.text());
if (!response.body) throw new Error('Missing response stream');

let summaryText = '';
let completed = false;

const readSummary = (payload) => {
  const result = payload.summary_result;
  if (typeof result === 'string' && result) return result;
  const overall = result?.summarization?.overall_summary;
  if (typeof overall === 'string' && overall) return overall;
  if (payload.metadata) {
    const nested = readSummary(payload.metadata);
    if (nested) return nested;
  }
  return payload.summary || payload.text;
};

const handleEvent = (event, data) => {
  const payload = JSON.parse(data.join('\n'));
  if (event === 'error' || payload.error || payload.response_state === 'failed') {
    throw new Error(payload.error || payload.status_message || 'Speech summarization failed');
  }

  if (event === 'transcribe_complete') {
    const transcript = typeof payload.transcript === 'string'
      ? JSON.parse(payload.transcript)
      : payload.transcript;
    const result = transcript.result || transcript;
    const text = result.text_display || result.transcript || result.text ||
      (result.segments || []).map((segment) =>
        typeof segment === 'string' ? segment : segment.text,
      ).join('\n');
    console.log(text);
    return;
  }

  if (event === 'done') {
    if (!summaryText.trim()) throw new Error('Stream completed without summary text');
    completed = true;
    return;
  }

  const text = readSummary(payload);
  if (typeof text === 'string' && text) {
    process.stdout.write(text.slice(summaryText.length));
    summaryText = text;
  }
};

const lines = createInterface({
  input: Readable.fromWeb(response.body),
  crlfDelay: Infinity,
});

let event = 'message';
let data = [];
for await (const line of lines) {
  if (!line) {
    if (data.length) handleEvent(event, data);
    event = 'message';
    data = [];
  } else if (line.startsWith('event:')) {
    event = line.slice(6).trim();
  } else if (line.startsWith('data:')) {
    data.push(line.slice(5).trimStart());
  }
}

if (data.length) handleEvent(event, data);
if (!completed) throw new Error('Stream ended before completion');
```
