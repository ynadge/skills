---
name: sapiom-browser
description: >
  Extract content from web pages as clean markdown or HTML, or capture
  screenshots of any URL using Anchor Browser via Sapiom's API — no Playwright,
  no Puppeteer, no infrastructure setup. Use when an agent needs to read a
  specific URL that requires JavaScript rendering, capture a visual snapshot of
  a page, or scrape content that Linkup/You.com web search can't reach. Prefer
  web search (sapiom-web-search) for general research — use browser automation
  only when you need a specific URL or visual capture.
license: MIT
metadata:
  version: "1.0.0"
---

# Browser Automation via Sapiom

Sapiom routes browser tasks through **Anchor Browser**, a headless cloud browser
with AI capabilities. Two operations: extract page content and capture
screenshots. Both are $0.01 flat per call.

## When to Apply This Skill

Use this skill when:
- Reading a specific URL that requires JavaScript to render (SPAs, dashboards)
- Capturing a screenshot for visual preview, testing, or monitoring
- Scraping content that requires a real browser (not just HTTP fetch)
- An agent needs to read a page whose content wasn't returned by web search

**Prefer `sapiom-web-search` for general research** — it's cheaper and returns
multiple results. Use browser automation when you need a specific page or
visual capture.

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
  serviceName: "Anchor Browser",
});

const BASE_URL = "https://anchor-browser.services.sapiom.ai/v1";
```

## Extract Page Content

Fetches a URL in a real browser and returns the main content as clean markdown
or raw HTML. Handles JavaScript-rendered pages automatically.

```typescript
const response = await fetch(`${BASE_URL}/tools/fetch-webpage`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    url: "https://example.com/blog/article",
    format: "markdown", // "markdown" or "html"
  }),
});

const data = await response.json();
console.log(data.title);   // page title
console.log(data.content); // clean markdown content
console.log(data.url);     // resolved URL (after any redirects)
```

### Handling JavaScript-Heavy Pages

For pages that load content after the initial HTML (SPAs, infinite scroll, etc.),
use the `wait` parameter to give JavaScript time to execute:

```typescript
body: JSON.stringify({
  url: "https://app.example.com/dashboard",
  format: "markdown",
  wait: 2000, // wait 2 seconds after page load before extracting
})
```

### Handling Timeouts

For slow pages, use `return_partial_on_timeout` to get whatever content loaded
rather than an error:

```typescript
body: JSON.stringify({
  url: "https://slow-site.example.com",
  format: "markdown",
  return_partial_on_timeout: true,
})
```

## Capture Screenshot

Takes a screenshot of a URL and returns it as a base64-encoded image.

```typescript
const response = await fetch(`${BASE_URL}/tools/screenshot`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    url: "https://example.com",
    width: 1280,
    height: 720,
    format: "png", // "png" or "jpeg"
  }),
});

const data = await response.json();
// data.image is "data:image/png;base64,iVBORw0KGgo..."
```

### Decoding the Base64 Image

The `image` field includes the data URI prefix — strip it before converting
to a buffer:

```typescript
import fs from "fs";

const data = await response.json();

// Strip the "data:image/png;base64," prefix before decoding
const base64Data = data.image.split(",")[1];
const imageBuffer = Buffer.from(base64Data, "base64");

fs.writeFileSync("screenshot.png", imageBuffer);
```

### Common Screenshot Sizes

```typescript
// Social preview / OG image
{ width: 1200, height: 630, format: "jpeg", image_quality: 90 }

// Full HD desktop
{ width: 1920, height: 1080, format: "png" }

// Mobile viewport
{ width: 390, height: 844, format: "png" }

// Full page capture (scrolls entire page)
{ width: 1280, height: 720, capture_full_height: true, scroll_all_content: true }
```

### Deprecated Parameters

The docs list two deprecated params — use the replacements instead:

| Deprecated | Use Instead |
|------------|-------------|
| `fullPage` | `capture_full_height: true` + `scroll_all_content: true` |
| `quality` | `image_quality` |

## Check Price Before Running

Both endpoints have a free `/price` variant:

```typescript
const priceResponse = await fetch(
  `${BASE_URL}/tools/screenshot/price`,
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ url: "https://example.com", width: 1280, height: 720 }),
  }
);
const { price } = await priceResponse.json(); // "$0.01"
```

Both operations are $0.01 flat, so the price check is mainly useful for
confirming the endpoint is reachable before kicking off a batch job.

## Reusable Functions

```typescript
import { createFetch } from "@sapiom/fetch";
import fs from "fs";

const fetch = createFetch({ apiKey: process.env.SAPIOM_API_KEY! });
const BASE_URL = "https://anchor-browser.services.sapiom.ai/v1";

// Extract page content as markdown
async function extractPage(
  url: string,
  options: { format?: "markdown" | "html"; wait?: number } = {}
): Promise<{ title: string; content: string; url: string }> {
  const response = await fetch(`${BASE_URL}/tools/fetch-webpage`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      url,
      format: options.format ?? "markdown",
      ...(options.wait && { wait: options.wait }),
    }),
  });

  if (!response.ok) {
    const status = response.status;
    if (status === 404) throw new Error(`Page not found or unreachable: ${url}`);
    if (status === 402) throw new Error("Sapiom credits exhausted");
    if (status === 429) throw new Error("Rate limited — back off and retry");
    throw new Error(`Extract failed: ${status}`);
  }

  return response.json();
}

// Capture screenshot and save to file
async function screenshotPage(
  url: string,
  outputPath: string,
  options: {
    width?: number;
    height?: number;
    format?: "png" | "jpeg";
    imageQuality?: number;
    fullPage?: boolean;
  } = {}
): Promise<Buffer> {
  const {
    width = 1280,
    height = 720,
    format = "png",
    imageQuality = 80,
    fullPage = false,
  } = options;

  const response = await fetch(`${BASE_URL}/tools/screenshot`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      url,
      width,
      height,
      format,
      image_quality: imageQuality,
      ...(fullPage && { capture_full_height: true, scroll_all_content: true }),
    }),
  });

  if (!response.ok) throw new Error(`Screenshot failed: ${response.status}`);

  const data = await response.json();
  const buffer = Buffer.from(data.image.split(",")[1], "base64");
  if (outputPath) fs.writeFileSync(outputPath, buffer);
  return buffer;
}

// Usage
const page = await extractPage("https://docs.example.com/api", { wait: 1000 });
console.log(page.title, page.content.slice(0, 200));

const img = await screenshotPage("https://example.com", "preview.png", {
  width: 1200,
  height: 630,
  format: "jpeg",
  imageQuality: 90,
});
```

## Error Handling

| Code | Meaning | What to do |
|------|---------|------------|
| 400 | Invalid URL or parameters | Ensure URL is valid and publicly accessible |
| 402 | Insufficient Sapiom credits | Top up at app.sapiom.ai |
| 404 | Page not found or unreachable | Check URL is publicly accessible; try `wait` for slow pages |
| 429 | Rate limited | Back off and retry with delay |

## Pricing

Both operations are flat-rate — no per-pixel or per-character billing.

| Operation | Price |
|-----------|-------|
| Extract (`fetch-webpage`) | $0.01 flat |
| Screenshot | $0.01 flat |

Full-page captures (`capture_full_height: true`) cost the same as viewport
screenshots — the flat rate covers any page length.

## Full API Reference

→ [references/browser_api.md](references/browser_api.md)