---
name: sapiom-images
description: >
  Generate images from text prompts, transform existing images with img2img,
  or upscale low-resolution images using FLUX and SDXL models via Sapiom's API —
  no FAL account, no vendor setup. Use when an agent needs to create, modify,
  or enhance images programmatically. Do NOT use for image analysis or
  understanding — this capability is generation only.
license: MIT
metadata:
  version: "1.0.0"
---

# Image Generation via Sapiom

Sapiom routes image requests through **FAL**, which provides fast inference for
FLUX and SDXL models. Three operations are supported: generate, transform, and
upscale.

## When to Apply This Skill

Use this skill when:
- An agent needs to generate an image from a text description
- Transforming an existing image into a new style (img2img)
- Upscaling a low-resolution image to higher resolution
- Building product visualization, content creation, or creative tooling

## Key Pattern: Model is in the URL

Unlike most APIs, the model is selected via the URL path, not a request body
parameter:

```
POST https://fal.services.sapiom.ai/v1/run/{model}
```

| Model | URL Path | Best For |
|-------|----------|----------|
| FLUX Dev | `fal-ai/flux/dev` | High quality, balanced speed |
| FLUX Schnell | `fal-ai/flux/schnell` | Fastest, cheapest, good quality |
| FLUX Pro | `fal-ai/flux-pro` | Highest quality, slowest, most expensive |
| SDXL | `fal-ai/stable-diffusion-xl` | Classic SDXL, good for stylized outputs |

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
  serviceName: "Fal.ai Image Generation",
});

const BASE_URL = "https://fal.services.sapiom.ai/v1";
```

## Generate an Image

```typescript
const response = await fetch(`${BASE_URL}/run/fal-ai/flux/dev`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    prompt: "A serene mountain lake at dawn with mist rising off the water",
    image_size: "landscape_16_9",
    num_images: 1,
  }),
});

const data = await response.json();
const imageUrl = data.images[0].url;
console.log(imageUrl); // https://fal.media/files/abc123.jpg
```

### Image Size Presets

| Preset | Dimensions | Megapixels |
|--------|------------|------------|
| `square_hd` | 1024 × 1024 | 1.05 MP |
| `square` | 512 × 512 | 0.26 MP |
| `portrait_4_3` | 768 × 1024 | 0.79 MP |
| `portrait_16_9` | 576 × 1024 | 0.59 MP |
| `landscape_4_3` | 1024 × 768 | 0.79 MP |
| `landscape_16_9` | 1024 × 576 | 0.59 MP |

Pricing is per megapixel — smaller presets cost less.

### Prompt Tips

- Be specific about lighting, style, and composition
- Add style qualifiers: `studio lighting`, `photorealistic`, `watercolor style`, `isometric`
- For product shots: `professional product photo, white background, studio lighting`
- Use `seed` for reproducible outputs across runs

```typescript
// Reproducible generation
body: JSON.stringify({
  prompt: "Minimalist desk lamp, product photo, white background",
  image_size: "square_hd",
  guidance_scale: 4.0,  // higher = closer to prompt, lower = more creative
  seed: 42,             // same seed + prompt = same image
})
```

## Transform an Existing Image

Modify an existing image using a text prompt. The `strength` parameter controls
how much the original image is changed — `0.0` keeps it identical, `1.0`
ignores it entirely.

```typescript
const response = await fetch(
  `${BASE_URL}/run/fal-ai/flux/dev/image-to-image`,
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      image_url: "https://example.com/photo.jpg", // must be a publicly accessible URL
      prompt: "Transform into a watercolor painting style",
      strength: 0.75, // 0.0 = no change, 1.0 = ignore original
      num_images: 1,
    }),
  }
);

const data = await response.json();
const transformedUrl = data.images[0].url;
```

**`strength` guide:**
- `0.3–0.5` — Subtle style changes, preserve original composition
- `0.6–0.8` — Moderate transformation, recognizable original
- `0.9–1.0` — Heavy transformation, original is just a loose reference

## Upscale an Image

Increase resolution 2x or 4x while preserving quality:

```typescript
const response = await fetch(
  `${BASE_URL}/run/fal-ai/topaz/upscale/image`,
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      image_url: "https://example.com/small-image.jpg",
      scale: 4, // 2 or 4
      output_format: "png", // jpeg or png
    }),
  }
);

const data = await response.json();
// Note: upscale returns data.image (singular), not data.images (array)
const upscaledUrl = data.image.url;
```

## Check Price Before Generating

Use the free price endpoint before running expensive generations. The path
mirrors the model path: `/v1/price/run/{model}` instead of `/v1/run/{model}`.

```typescript
const priceResponse = await fetch(
  `${BASE_URL}/price/run/fal-ai/flux-pro`,
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      prompt: "...",
      image_size: "square_hd",
    }),
  }
);

const { price } = await priceResponse.json();
// e.g. "$0.042" — decide whether to use flux-pro or drop down to flux/dev
```

## Reusable Generation Function

```typescript
import { createFetch } from "@sapiom/fetch";

const fetch = createFetch({ apiKey: process.env.SAPIOM_API_KEY! });
const BASE_URL = "https://fal.services.sapiom.ai/v1";

type ImageSize =
  | "square_hd" | "square"
  | "portrait_4_3" | "portrait_16_9"
  | "landscape_4_3" | "landscape_16_9";

type FluxModel = "fal-ai/flux/schnell" | "fal-ai/flux/dev" | "fal-ai/flux-pro" | "fal-ai/stable-diffusion-xl";

async function generateImage(
  prompt: string,
  options: {
    model?: FluxModel;
    size?: ImageSize;
    numImages?: number;
    seed?: number;
  } = {}
): Promise<string[]> {
  const {
    model = "fal-ai/flux/dev",
    size = "landscape_4_3",
    numImages = 1,
    seed,
  } = options;

  const response = await fetch(`${BASE_URL}/run/${model}`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      prompt,
      image_size: size,
      num_images: numImages,
      ...(seed !== undefined && { seed }),
    }),
  });

  if (!response.ok) {
    const status = response.status;
    if (status === 402) throw new Error("Sapiom credits exhausted");
    if (status === 413) throw new Error("Input image too large");
    if (status === 429) throw new Error("Rate limited — back off and retry");
    throw new Error(`Generation failed: ${status}`);
  }

  const data = await response.json();
  return data.images.map((img: any) => img.url);
}

// Usage
const urls = await generateImage("A cozy coffee shop on a rainy day", {
  model: "fal-ai/flux/schnell", // fast and cheap for drafts
  size: "landscape_16_9",
});
```

## Error Handling

| Code | Meaning | What to do |
|------|---------|------------|
| 400 | Invalid parameters or bad image URL | Check image URL is publicly accessible; verify size preset spelling |
| 402 | Insufficient Sapiom credits | Top up at app.sapiom.ai |
| 404 | Model not found | Check the model path in the URL |
| 413 | Image too large | Resize input image before sending to transform/upscale |
| 429 | Rate limited | Back off and retry with delay |

## Pricing

Pricing is per **megapixel (MP)** of the output image. `landscape_4_3` = 0.79 MP,
`square_hd` = 1.05 MP.

| Model / Operation | Price per MP | Example: `square_hd` |
|-------------------|--------------|----------------------|
| `flux/schnell` | $0.004/MP | ~$0.004 |
| `flux/dev` | $0.015/MP | ~$0.016 |
| `flux-pro` | $0.040/MP | ~$0.042 |
| `stable-diffusion-xl` | $0.007/MP | ~$0.007 |
| Transform (img2img) | $0.045/MP | ~$0.047 |
| Upscale | $0.030/MP | ~$0.032 |

**Model selection guidance:**
- Use `flux/schnell` for drafts, prototyping, or high-volume generation
- Use `flux/dev` as the default for production quality
- Use `flux-pro` only when quality is critical — it's 10x the cost of schnell
- Transform and upscale are the most expensive per-MP operations — use the
  price endpoint before running them on large images