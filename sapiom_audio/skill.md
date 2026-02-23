---
name: sapiom-audio
description: >
  Convert text to natural-sounding speech or generate sound effects using
  ElevenLabs via Sapiom's API — no ElevenLabs account, no vendor setup. Use
  when an agent needs to produce spoken audio from text (TTS) or create
  sound effects from a text description. Do NOT use for speech-to-text
  transcription — this capability is audio generation only.
license: MIT
metadata:
  version: "1.0.0"
---

# Audio Services via Sapiom

Sapiom routes audio requests through **ElevenLabs**, providing natural-sounding
TTS across 29 languages and AI-generated sound effects. Two operations:
text-to-speech and sound effects generation.

## When to Apply This Skill

Use this skill when:
- An agent needs to speak a response aloud (voice assistants, accessibility)
- Generating narration, podcast intros, or voiceovers programmatically
- Creating custom sound effects for UI, games, or media production
- Building any application that needs audio output without ElevenLabs setup

## Critical: Responses Are Binary Audio

Unlike most Sapiom endpoints, TTS and sound effects return **raw binary audio
data**, not JSON. You must handle the response as an `ArrayBuffer`, not call
`.json()` on it.

```typescript
// ❌ Wrong — will throw or produce garbage
const data = await response.json();

// ✅ Correct
const buffer = await response.arrayBuffer();
const audioBytes = Buffer.from(buffer);
```

## Setup

```bash
npm install @sapiom/fetch
```

```bash
export SAPIOM_API_KEY="your_api_key_here"
```

```typescript
import { createFetch } from "@sapiom/fetch";

const fetch = createFetch({
  apiKey: process.env.SAPIOM_API_KEY!,
  agentName: "my-agent",
  serviceName: "ElevenLabs TTS",
});

const BASE_URL = "https://elevenlabs.services.sapiom.ai";
```

## Text-to-Speech

The voice ID is specified in the URL path. The response is binary MP3 audio.

```typescript
import fs from "fs";

const response = await fetch(
  `${BASE_URL}/v1/text-to-speech/EXAVITQu4vr4xnSDxMaL`, // Sarah voice
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      text: "Welcome to our application. How can I help you today?",
      model_id: "eleven_multilingual_v2",
    }),
  }
);

const buffer = Buffer.from(await response.arrayBuffer());
fs.writeFileSync("output.mp3", buffer);

// Character count is available in response headers
const charCount = response.headers.get("X-Character-Count");
```

### Popular Voices

| Voice ID | Name | Character |
|----------|------|-----------|
| `EXAVITQu4vr4xnSDxMaL` | Sarah | Female, soft |
| `JBFqnCBsd6RMkjVDRZzb` | George | Male, narrative |
| `21m00Tcm4TlvDq8ikWAM` | Rachel | Female, calm |
| `AZnzlk1XvdvUeBnXmlld` | Domi | Female, strong |

To get the full list of available voices (free, no payment):

```typescript
const response = await fetch(`${BASE_URL}/v2/voices`);
const { voices } = await response.json();
voices.forEach((v: any) => console.log(`${v.name}: ${v.voice_id}`));
```

### Output Format Options

Default is `mp3_44100_128` — good quality at reasonable file size. Pass
`output_format` in the request body to override.

| Format | Use When |
|--------|----------|
| `mp3_44100_128` | Default — good quality, wide compatibility |
| `mp3_44100_192` | Higher quality MP3 |
| `mp3_22050_32` | Smallest file size, lower quality |
| `pcm_24000` | Streaming or real-time playback |
| `opus_48000_128` | Web streaming (WebRTC, browser audio) |

```typescript
body: JSON.stringify({
  text: "...",
  model_id: "eleven_multilingual_v2",
  output_format: "pcm_24000", // for streaming
})
```

## Sound Effects

Generate a sound effect from a text description. Response is binary MP3 audio.

```typescript
const response = await fetch(`${BASE_URL}/v1/sound-generation`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    text: "Cinematic braam, horror atmosphere",
    duration_seconds: 3.0,     // 0.5–22.0 seconds
    prompt_influence: 0.5,     // 0.0 = loose, 1.0 = literal
  }),
});

const buffer = Buffer.from(await response.arrayBuffer());
fs.writeFileSync("sfx.mp3", buffer);
```

### Sound Effect Prompt Tips

- Be specific about genre and mood: `"cinematic braam, tension building"`
- Describe the environment: `"forest ambience, birds, light wind, no music"`
- Reference familiar categories: `"UI click, soft, modern app style"`
- `prompt_influence` at `0.3` (default) gives creative interpretation;
  raise to `0.7+` for more literal results

## Check Price Before Generating

Both endpoints have a free `/price` variant — append `/price` to the path:

```typescript
// TTS price check
const priceResponse = await fetch(
  `${BASE_URL}/v1/text-to-speech/EXAVITQu4vr4xnSDxMaL/price`,
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      text: longScript,
      model_id: "eleven_multilingual_v2",
    }),
  }
);
const { price } = await priceResponse.json();
// e.g. "$0.12" for a 500-character script
```

## Reusable Functions

```typescript
import { createFetch } from "@sapiom/fetch";
import fs from "fs";

const fetch = createFetch({ apiKey: process.env.SAPIOM_API_KEY! });
const BASE_URL = "https://elevenlabs.services.sapiom.ai";

async function textToSpeech(
  text: string,
  options: {
    voiceId?: string;
    outputFormat?: string;
    outputPath?: string;
  } = {}
): Promise<Buffer> {
  const {
    voiceId = "EXAVITQu4vr4xnSDxMaL", // Sarah
    outputFormat = "mp3_44100_128",
    outputPath,
  } = options;

  if (text.length > 5000) {
    throw new Error("Text exceeds 5000 character limit — split into chunks");
  }

  const response = await fetch(
    `${BASE_URL}/v1/text-to-speech/${voiceId}`,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        text,
        model_id: "eleven_multilingual_v2",
        output_format: outputFormat,
      }),
    }
  );

  if (!response.ok) {
    const status = response.status;
    if (status === 402) throw new Error("Sapiom credits exhausted");
    if (status === 413) throw new Error("Text too large — max 5000 characters");
    if (status === 404) throw new Error(`Voice not found: ${voiceId}`);
    throw new Error(`TTS failed: ${status}`);
  }

  const audio = Buffer.from(await response.arrayBuffer());
  if (outputPath) fs.writeFileSync(outputPath, audio);
  return audio;
}

async function generateSoundEffect(
  description: string,
  durationSeconds: number = 2.0
): Promise<Buffer> {
  const response = await fetch(`${BASE_URL}/v1/sound-generation`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      text: description,
      duration_seconds: durationSeconds,
      prompt_influence: 0.3,
    }),
  });

  if (!response.ok) throw new Error(`Sound generation failed: ${response.status}`);
  return Buffer.from(await response.arrayBuffer());
}

// Usage
const speech = await textToSpeech("Hello, world!", {
  voiceId: "JBFqnCBsd6RMkjVDRZzb", // George
  outputPath: "speech.mp3",
});

const sfx = await generateSoundEffect("soft UI notification chime", 1.0);
fs.writeFileSync("chime.mp3", sfx);
```

## Error Handling

| Code | Meaning | What to do |
|------|---------|------------|
| 400 | Invalid parameters | Check text length (max 5000 chars); check duration range (0.5–22s) |
| 402 | Insufficient Sapiom credits | Top up at app.sapiom.ai |
| 404 | Voice or model not found | Check voice ID against `GET /v2/voices` |
| 413 | Text too large | Split into smaller chunks and concatenate audio |
| 429 | Rate limited | Back off and retry with delay |

## Pricing

| Operation | Price | Unit |
|-----------|-------|------|
| Text-to-Speech | $0.24 | per 1,000 characters |
| Sound Effects | $0.08 | flat per generation |

**Examples:**
- 100-char TTS (~one short sentence): ~$0.024
- 500-char TTS (~two paragraphs): ~$0.12
- Sound effect (any duration): $0.08 flat

TTS is priced per character — keep text concise to control cost. Sound effects
are always $0.08 regardless of duration, so generating a 22-second effect costs
the same as a 0.5-second one.