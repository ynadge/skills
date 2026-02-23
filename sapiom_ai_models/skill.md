---
name: sapiom-ai-models
description: >
  Access 400+ AI models (GPT-4o, Claude, Gemini, Llama, Mistral, and more)
  through a single Sapiom API key — no separate OpenAI, Anthropic, or Google
  accounts required. Use when an agent needs to call an LLM for chat completions,
  structured JSON output, function/tool calling, or text embeddings. Also use
  when switching between models without changing integration code. Do NOT use
  when you already have direct API access to a model — Sapiom adds cost on top
  of model pricing, so only use it when the zero-vendor-setup benefit applies.
license: MIT
metadata:
  version: "1.0.0"
---

# AI Model Access via Sapiom

Sapiom routes model requests through **OpenRouter**, which aggregates 400+ models
from OpenAI, Anthropic, Google, Meta, Mistral, and others behind a single
OpenAI-compatible API. One key, any model.

## When to Apply This Skill

Use this skill when:
- An agent needs to call an LLM without managing separate provider accounts
- Switching between models based on task (cost vs capability tradeoffs)
- Building applications that need model flexibility without re-integration
- Generating text embeddings for RAG or semantic search
- Using function/tool calling or JSON mode on supported models

## Critical: Two Common Gotchas

Before writing any code, know these — both cause confusing errors:

**1. Always include `max_tokens`**
The gateway uses `max_tokens` to pre-authorize cost. Omitting it will fail.

```typescript
// ❌ Wrong — missing max_tokens
{ model: "openai/gpt-4o-mini", messages: [...] }

// ✅ Correct
{ model: "openai/gpt-4o-mini", messages: [...], max_tokens: 500 }
```

**2. Always use the full `provider/model` name**
```typescript
// ❌ Wrong
{ model: "gpt-4o-mini" }

// ✅ Correct
{ model: "openai/gpt-4o-mini" }
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
  serviceName: "OpenRouter Chat",
});

const BASE_URL = "https://openrouter.services.sapiom.ai/v1";
```

## Chat Completions (OpenAI format)

The primary endpoint. Works with any model on OpenRouter using the standard
OpenAI chat completions format.

```typescript
const response = await fetch(`${BASE_URL}/chat/completions`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "openai/gpt-4o-mini",
    messages: [
      { role: "system", content: "You are a helpful assistant." },
      { role: "user", content: "Explain TypeScript generics briefly." },
    ],
    max_tokens: 500,
    temperature: 0.7,
  }),
});

const data = await response.json();
console.log(data.choices[0].message.content);
console.log(`Tokens used: ${data.usage.total_tokens}`);
```

### JSON Mode (structured output)

Force the model to respond with valid JSON:

```typescript
const response = await fetch(`${BASE_URL}/chat/completions`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "openai/gpt-4o-mini",
    messages: [
      {
        role: "user",
        content: "List 3 programming languages as JSON with name and useCase fields.",
      },
    ],
    max_tokens: 300,
    response_format: { type: "json_object" },
  }),
});

const data = await response.json();
const parsed = JSON.parse(data.choices[0].message.content);
```

### Tool / Function Calling

```typescript
const response = await fetch(`${BASE_URL}/chat/completions`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "openai/gpt-4o",
    messages: [{ role: "user", content: "What's the weather in Paris?" }],
    max_tokens: 200,
    tools: [
      {
        type: "function",
        function: {
          name: "get_weather",
          description: "Get current weather for a city",
          parameters: {
            type: "object",
            properties: {
              city: { type: "string", description: "City name" },
            },
            required: ["city"],
          },
        },
      },
    ],
  }),
});

const data = await response.json();
const toolCall = data.choices[0].message.tool_calls?.[0];
if (toolCall) {
  const args = JSON.parse(toolCall.function.arguments);
  console.log("Model wants to call:", toolCall.function.name, args);
}
```

## Anthropic Messages Format

If you're already using Anthropic's SDK patterns, use this endpoint instead —
it accepts the native Anthropic format and the gateway translates automatically.

```typescript
const response = await fetch(`${BASE_URL}/messages`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "anthropic/claude-3.5-sonnet",
    max_tokens: 500,
    messages: [
      { role: "user", content: "Explain TypeScript generics." },
    ],
  }),
});
```

## Embeddings

Generate vector embeddings for RAG pipelines, semantic search, or similarity
scoring:

```typescript
const response = await fetch(`${BASE_URL}/embeddings`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "openai/text-embedding-3-small",
    input: "The quick brown fox jumps over the lazy dog",
  }),
});

const data = await response.json();
const vector = data.data[0].embedding; // float[]
```

## Choosing a Model

Use the free `/v1/models` endpoint to get the full list with current pricing:

```typescript
const response = await fetch(`${BASE_URL}/models`);
const { data: models } = await response.json();
// models is an array of { id, pricing, context_length, ... }
```

For quick reference, see the curated model table in:
→ [references/models-api.md](references/models-api.md)

**General guidance:**
- **Cost-sensitive tasks** (classification, summarization, extraction): `openai/gpt-4o-mini` or `google/gemini-flash`
- **Complex reasoning** (code generation, analysis, planning): `openai/gpt-4o` or `anthropic/claude-3.5-sonnet`
- **Long context** (large documents, long conversations): `google/gemini-pro` or `meta-llama/llama-3.1-405b`
- **Embeddings**: `openai/text-embedding-3-small` (fast/cheap) or `openai/text-embedding-3-large` (higher accuracy)

## Check Price Before Running

Every endpoint has a free `/price` variant. Use it before expensive calls:

```typescript
const priceResponse = await fetch(`${BASE_URL}/chat/completions/price`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "anthropic/claude-3-opus",
    messages: [{ role: "user", content: "..." }],
    max_tokens: 2000,
  }),
});

const { price } = await priceResponse.json();
console.log(`Max cost for this call: ${price}`);
// Decide whether to proceed or switch to a cheaper model
```

## Reusable Chat Function

```typescript
import { createFetch } from "@sapiom/fetch";

const fetch = createFetch({ apiKey: process.env.SAPIOM_API_KEY! });
const BASE_URL = "https://openrouter.services.sapiom.ai/v1";

type Message = { role: "system" | "user" | "assistant"; content: string };

async function chat(
  model: string,
  messages: Message[],
  options: { maxTokens?: number; temperature?: number; jsonMode?: boolean } = {}
): Promise<string> {
  const { maxTokens = 500, temperature = 0.7, jsonMode = false } = options;

  const response = await fetch(`${BASE_URL}/chat/completions`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      model,
      messages,
      max_tokens: maxTokens,
      temperature,
      ...(jsonMode && { response_format: { type: "json_object" } }),
    }),
  });

  if (!response.ok) {
    const status = response.status;
    if (status === 404) throw new Error(`Model not found: ${model} — use full provider/model format`);
    if (status === 402) throw new Error("Sapiom credits exhausted — top up at app.sapiom.ai");
    if (status === 429) throw new Error("Rate limited — back off and retry");
    throw new Error(`Request failed: ${status}`);
  }

  const data = await response.json();
  return data.choices[0].message.content;
}

// Usage
const reply = await chat(
  "openai/gpt-4o-mini",
  [
    { role: "system", content: "You are a helpful assistant." },
    { role: "user", content: "What is a closure in JavaScript?" },
  ],
  { maxTokens: 300 }
);
```

## Error Handling

| Code | Meaning | What to do |
|------|---------|------------|
| 400 | Invalid request | Check model name format and that `max_tokens` is set |
| 402 | Insufficient Sapiom credits | Top up at app.sapiom.ai |
| 404 | Model not found | Use full `provider/model` format — check `/v1/models` for valid IDs |
| 429 | Rate limited | Exponential backoff and retry |

## Pricing

Pricing is per-token and varies by model. You authorize a maximum cost based
on `max_tokens` but only pay for tokens actually generated.

Check per-model rates at [OpenRouter](https://openrouter.ai/docs#models) or
use the free `/v1/models` endpoint. Use the `/price` endpoints before any
expensive call.

## Full API & Model Reference

→ [references/models_api.md](references/models_api.md)