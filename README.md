# Sapiom Skills

Agent skills for [Sapiom](https://sapiom.ai) — instant access to paid services for AI agents.

## Installation

```bash
# Install the overview skill
npx skills add sapiom/skills --skill sapiom

# Install a specific capability skill
npx skills add sapiom/skills --skill sapiom-web-search
npx skills add sapiom/skills --skill sapiom-ai-models
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `sapiom` | SDK setup, service endpoints, and pricing for Sapiom capabilities |
| `sapiom-web-search` | Web search via Linkup and You.com with sourced answers, structured extraction, and URL fetching |
| `sapiom-verify` | Phone and email OTP verification via Prelude (send code, check code) |
| `sapiom-ai-models` | Access 400+ LLMs (GPT-4o, Claude, Gemini, Llama) for chat completions, embeddings, and tool calling |
| `sapiom-images` | Image generation, transformation, and upscaling via FLUX and SDXL models |
| `sapiom-audio` | Text-to-speech and sound effects generation via ElevenLabs |
| `sapiom-browser` | Page content extraction and screenshots via headless browser automation |

## Links

- [Sapiom Documentation](https://docs.sapiom.ai)
- [Agent Skills Format](https://agentskills.io)
