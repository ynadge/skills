# AI Models API Reference

Full endpoint reference and curated model guide for Sapiom's AI model access capability.

**Base URL:** `https://openrouter.services.sapiom.ai/v1`

---

## Endpoints

| Method | Path | Description | Free? |
|--------|------|-------------|-------|
| POST | `/chat/completions` | Chat completion (OpenAI format) | No |
| POST | `/chat/completions/price` | Price estimate for chat completion | Yes |
| POST | `/messages` | Chat completion (Anthropic format) | No |
| POST | `/messages/price` | Price estimate for messages | Yes |
| POST | `/responses` | Chat completion (OpenAI Responses format) | No |
| POST | `/responses/price` | Price estimate for responses | Yes |
| POST | `/embeddings` | Generate text embeddings | No |
| POST | `/embeddings/price` | Price estimate for embeddings | Yes |
| GET | `/models` | List all available models and pricing | Yes |

---

## POST /chat/completions

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | Yes | Full model ID with provider prefix (e.g. `openai/gpt-4o-mini`) |
| `messages` | array | Yes | Conversation history — array of `{ role, content }` objects |
| `max_tokens` | number | **Yes** | Max tokens to generate — required for cost pre-authorization |
| `temperature` | number | No | Sampling temperature 0–2 (default: 1) |
| `top_p` | number | No | Nucleus sampling 0–1 |
| `frequency_penalty` | number | No | Penalize token frequency -2 to 2 |
| `presence_penalty` | number | No | Penalize token presence -2 to 2 |
| `stop` | string[] | No | Stop sequences |
| `tools` | array | No | Tool definitions for function calling |
| `tool_choice` | string/object | No | `auto`, `none`, or `{ type: "function", function: { name } }` |
| `response_format` | object | No | `{ "type": "json_object" }` to enable JSON mode |
| `stream` | boolean | No | Stream partial responses (default: false) |

### Message Roles

| Role | Description |
|------|-------------|
| `system` | Sets model behavior and context |
| `user` | Human turn |
| `assistant` | Model turn (for multi-turn conversations) |
| `tool` | Tool result (when using function calling) |

### Response

```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1700000000,
  "model": "openai/gpt-4o-mini",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "string",
        "tool_calls": []   // present when model calls a tool
      },
      "finish_reason": "stop"  // stop | length | tool_calls | content_filter
    }
  ],
  "usage": {
    "prompt_tokens": 25,
    "completion_tokens": 150,
    "total_tokens": 175
  }
}
```

---

## POST /embeddings

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | Yes | Embedding model ID (e.g. `openai/text-embedding-3-small`) |
| `input` | string or string[] | Yes | Text to embed — single string or batch array |

### Response

```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "index": 0,
      "embedding": [0.0023, -0.0085, ...]  // float[]
    }
  ],
  "usage": {
    "prompt_tokens": 8,
    "total_tokens": 8
  }
}
```

---

## GET /models

Returns all available models with pricing. Free, no auth required beyond SDK.

### Response (per model)

```json
{
  "id": "openai/gpt-4o-mini",
  "name": "GPT-4o Mini",
  "context_length": 128000,
  "pricing": {
    "prompt": "0.00000015",    // per token
    "completion": "0.0000006"  // per token
  }
}
```

---

## Curated Model Reference

A practical guide to the most useful models available. For the full list, call `GET /v1/models`.

### Recommended by Use Case

#### Fast & Cost-Efficient (everyday tasks)
Best for classification, summarization, extraction, Q&A, simple generation.

| Model ID | Context | Notes |
|----------|---------|-------|
| `openai/gpt-4o-mini` | 128k | Best default for cost-sensitive tasks |
| `google/gemini-flash` | 1M | Extremely fast, very cheap, huge context |
| `mistralai/mistral-small` | 32k | Strong European option, GDPR-friendly |

#### High Capability (complex tasks)
Best for code generation, complex reasoning, multi-step planning, nuanced writing.

| Model ID | Context | Notes |
|----------|---------|-------|
| `openai/gpt-4o` | 128k | Top OpenAI model, strong at code and reasoning |
| `anthropic/claude-3.5-sonnet` | 200k | Excellent at analysis, writing, and long docs |
| `anthropic/claude-3-opus` | 200k | Highest capability Anthropic model, highest cost |
| `google/gemini-pro` | 1M | Best for tasks needing massive context |

#### Long Context
Best for large codebases, long documents, extended conversations.

| Model ID | Context | Notes |
|----------|---------|-------|
| `google/gemini-flash` | 1M | 1 million token context at low cost |
| `google/gemini-pro` | 1M | 1 million token context, higher capability |
| `anthropic/claude-3.5-sonnet` | 200k | 200k context, excellent at following long docs |
| `meta-llama/llama-3.1-405b` | 128k | Large open-weight model, good for long tasks |

#### Open Weight (self-hostable, no data sharing)

| Model ID | Context | Notes |
|----------|---------|-------|
| `meta-llama/llama-3.1-405b` | 128k | Largest Llama model, near frontier quality |
| `meta-llama/llama-3.2-90b` | 128k | Smaller Llama, faster and cheaper |
| `mistralai/mixtral-8x7b` | 32k | Strong MoE model, very cost-efficient |
| `mistralai/mistral-large` | 32k | Mistral's top model |

#### Embeddings

| Model ID | Dimensions | Notes |
|----------|------------|-------|
| `openai/text-embedding-3-small` | 1536 | Best default — fast, cheap, high quality |
| `openai/text-embedding-3-large` | 3072 | Higher accuracy, use when similarity matters most |

---

## Error Codes

| Code | Meaning | Action |
|------|---------|--------|
| 400 | Invalid request | Verify `max_tokens` is set; check `model` uses `provider/name` format |
| 402 | Insufficient Sapiom credits | Top up at app.sapiom.ai |
| 404 | Model not found | Check exact model ID against `GET /v1/models` |
| 429 | Rate limited | Exponential backoff and retry |

---

## Pricing Notes

- Pricing is per token, billed on actual usage (not `max_tokens`)
- You pre-authorize up to `max_tokens` output cost, but only pay for what's generated
- Use `/price` endpoints before expensive calls — they're always free
- Cheapest models (`gpt-4o-mini`, `gemini-flash`) cost ~$0.15–$0.30 per 1M input tokens
- Most expensive models (`claude-3-opus`, `gpt-4o`) cost ~$15–$30 per 1M input tokens
- See live pricing: `GET /v1/models` or [openrouter.ai/docs#models](https://openrouter.ai/docs#models)