# AI Integration

Build real-time AI features for Zoom meetings - sentiment analysis, summarization, and more.

## Overview

Integrate AI/ML capabilities with Zoom meetings using real-time media streams for live transcription, sentiment analysis, meeting summarization, and intelligent automation.

## Skills Needed

- **rtms** - Primary (real-time media access)
- **zoom-meeting-sdk** (Linux) - For meeting bots
- **scribe** - File or archive transcription after media capture/export
- **summarizer** - Transcript recap, summary, and action item extraction
- **translator** - Plain text and transcript translation

## AI Use Cases

| Use Case | Input | Output |
|----------|-------|--------|
| Transcription | Audio stream | Real-time text |
| Sentiment | Audio/transcript | Mood indicators |
| Summarization | Transcript | Meeting summary or action items |
| Action items | Transcript | Task list |
| Translation | Audio/transcript | Multi-language |

## Architecture

```
AI Integration Architecture:
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    Zoom     │────▶│   RTMS /    │────▶│  AI/ML      │
│   Meeting   │     │   Bot SDK   │     │  Pipeline   │
└─────────────┘     └─────────────┘     └─────────────┘
                           │
                    Audio/Video/Transcript
```

## Prerequisites

- RTMS access or Meeting SDK (Linux)
- AI/ML service (OpenAI, Azure, custom)
- Real-time processing infrastructure

## Common Tasks

### Setting Up RTMS for AI

```javascript
// 1. Configure RTMS app in Marketplace
// Enable: Audio stream, Video stream, Transcript stream

// 2. Handle webhook to get connection details
app.post('/webhook', (req, res) => {
  if (req.body.event === 'meeting.rtms_started') {
    const { server_urls, stream_id, signature } = req.body.payload;
    
    // Start AI processing pipeline
    aiPipeline.connect({
      url: server_urls[0],
      streamId: stream_id,
      signature: signature
    });
  }
  res.status(200).send();
});
```

### Real-Time Transcription Pipeline

```javascript
// Option 1: Use Zoom's built-in transcript (via RTMS)
rtmsClient.on('transcript', (data) => {
  const { text, speaker_id, is_final } = data;
  if (is_final) {
    transcriptStore.append(speaker_id, text);
  }
});

// Option 2: Send audio to external STT (Whisper, Deepgram)
const deepgram = new Deepgram(DEEPGRAM_KEY);
const transcriber = deepgram.transcription.live({
  punctuate: true,
  interim_results: true,
  language: 'en-US'
});

rtmsClient.on('audio', (audioChunk) => {
  transcriber.send(audioChunk);
});

transcriber.on('transcriptReceived', (data) => {
  const transcript = data.channel.alternatives[0].transcript;
  processTranscript(transcript);
});
```

### Sentiment Analysis Integration

```javascript
// Real-time sentiment on transcript segments
async function analyzeSentiment(text) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [{
      role: 'system',
      content: 'Analyze sentiment. Return JSON: {sentiment: "positive|negative|neutral", confidence: 0-1, emotions: []}'
    }, {
      role: 'user',
      content: text
    }],
    response_format: { type: 'json_object' }
  });
  
  return JSON.parse(response.choices[0].message.content);
}

// Track sentiment over time
class SentimentTracker {
  constructor() {
    this.history = [];
  }
  
  async process(transcript) {
    const sentiment = await analyzeSentiment(transcript);
    this.history.push({
      timestamp: Date.now(),
      text: transcript,
      ...sentiment
    });
    
    // Alert on negative sentiment
    if (sentiment.sentiment === 'negative' && sentiment.confidence > 0.8) {
      this.emit('alert', { type: 'negative_sentiment', data: sentiment });
    }
  }
  
  getOverallSentiment() {
    // Aggregate sentiment over meeting duration
  }
}
```

### Meeting Summarization

For Zoom-native transcript summarization, use [../../summarizer/SKILL.md](../../summarizer/SKILL.md):

- fast mode: `POST /aiservices/summarizer/summarize`
- tasks: `recap`, `summary`, `action_items`, `full_summary`
- batch mode: summarize stored `.vtt`, `.srt`, or `.txt` transcript files

For text localization, use [../../translator/SKILL.md](../../translator/SKILL.md):

- fast mode: `POST /aiservices/translator/translate`
- batch mode: translate stored text files
- guardrail: one target language per request/job, with English as source or target

### Latency Considerations

| Processing Type | Target Latency | Recommendation |
|-----------------|----------------|----------------|
| Live captions | < 500ms | Use streaming STT (Deepgram, AssemblyAI) |
| Sentiment | < 2s | Batch every 10-15 seconds |
| Summarization | Post-meeting | Process after meeting ends |
| Action items | < 5s | Process paragraph by paragraph |

### Example AI Pipeline Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    RTMS WebSocket                       │
└─────────────┬───────────────┬───────────────┬──────────┘
              │               │               │
         Audio Stream    Video Stream    Transcript
              │               │               │
              ▼               ▼               ▼
      ┌───────────────┐ ┌──────────┐ ┌───────────────┐
      │ Speech-to-Text│ │Face/OCR  │ │ NLP Pipeline  │
      │ (Deepgram)    │ │Detection │ │ (OpenAI GPT)  │
      └───────┬───────┘ └────┬─────┘ └───────┬───────┘
              │              │               │
              ▼              ▼               ▼
      ┌─────────────────────────────────────────────────┐
      │              Results Aggregator                  │
      │  - Transcripts  - Sentiment  - Action Items     │
      └─────────────────────┬───────────────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   Storage /   │
                    │   Dashboard   │
                    └───────────────┘
```

## Resources

- **RTMS docs**: https://developers.zoom.us/docs/rtms/
- **Meeting SDK Linux**: https://developers.zoom.us/docs/meeting-sdk/linux/
- **Deepgram**: https://deepgram.com/
- **OpenAI API**: https://platform.openai.com/docs
