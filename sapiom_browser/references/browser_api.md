# Browser API Reference

Full parameter reference for Sapiom's browser automation capability.

**Base URL:** `https://anchor-browser.services.sapiom.ai/v1`

---

## Endpoints

| Method | Path | Description | Free? |
|--------|------|-------------|-------|
| POST | `/tools/fetch-webpage` | Extract page content as markdown or HTML | No |
| POST | `/tools/fetch-webpage/price` | Price estimate for extraction | Yes |
| POST | `/tools/screenshot` | Capture page screenshot | No |
| POST | `/tools/screenshot/price` | Price estimate for screenshot | Yes |

---

## POST /tools/fetch-webpage

Loads a URL in a real headless browser and extracts the main content.
Handles JavaScript rendering, redirects, and dynamic content.

### Request Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `url` | string | Yes | — | URL to fetch |
| `format` | string | No | `markdown` | Output format: `markdown` or `html` |
| `wait` | number | No | — | Milliseconds to wait after page load before extracting (for JS-heavy pages) |
| `new_page` | boolean | No | `false` | Open URL in a new browser page |
| `page_index` | number | No | — | Target page index in a multi-page session |
| `return_partial_on_timeout` | boolean | No | `false` | Return partial content on timeout instead of erroring |

### Response

```json
{
  "content": "# Page Title\n\nExtracted content in markdown...",
  "title": "Page Title",
  "url": "https://example.com/final-url"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `content` | string | Extracted content in the requested format |
| `title` | string | Page `<title>` tag value |
| `url` | string | Resolved URL after redirects |

---

## POST /tools/fetch-webpage/price

Free endpoint — returns cost estimate before extraction. Accepts the same
parameters as `/tools/fetch-webpage`.

### Response

```json
{
  "price": "$0.01",
  "currency": "USD"
}
```

---

## POST /tools/screenshot

Loads a URL in a headless browser and captures a screenshot.
Returns a base64-encoded image.

### Request Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `url` | string | Yes | — | URL to screenshot |
| `width` | number | No | `1280` | Viewport width in pixels (320–3840) |
| `height` | number | No | `720` | Viewport height in pixels (240–2160) |
| `format` | string | No | `png` | Image format: `png` or `jpeg` |
| `image_quality` | number | No | `80` | JPEG quality (1–100); ignored for PNG |
| `wait` | number | No | — | Milliseconds to wait after page load before capturing |
| `capture_full_height` | boolean | No | `false` | Extend capture to full page height beyond viewport |
| `scroll_all_content` | boolean | No | `false` | Scroll through page before capture (triggers lazy-loaded content) |
| `s3_target_address` | string | No | — | S3 URL for direct upload; if set, `image` field in response will be empty |

**Deprecated parameters — do not use:**

| Deprecated | Replacement |
|------------|-------------|
| `fullPage` | `capture_full_height: true` + `scroll_all_content: true` |
| `quality` | `image_quality` |

### Response

```json
{
  "image": "data:image/png;base64,iVBORw0KGgo...",
  "width": 1280,
  "height": 720
}
```

| Field | Type | Description |
|-------|------|-------------|
| `image` | string | Base64 data URI — includes `data:image/{format};base64,` prefix |
| `width` | number | Actual captured width in pixels |
| `height` | number | Actual captured height in pixels |

**Important:** The `image` value includes the data URI prefix. Strip it before
decoding to a buffer:

```typescript
const base64Data = data.image.split(",")[1]; // remove "data:image/png;base64,"
const buffer = Buffer.from(base64Data, "base64");
```

If `s3_target_address` is provided, the image is uploaded directly to S3 and
`image` will be empty in the response.

---

## POST /tools/screenshot/price

Free endpoint — returns cost estimate before capturing. Accepts the same
parameters as `/tools/screenshot`.

### Response

```json
{
  "price": "$0.01",
  "currency": "USD"
}
```

---

## Error Codes

| Code | Meaning | Action |
|------|---------|--------|
| 400 | Invalid URL or bad parameters | Verify URL format and parameter ranges |
| 402 | Insufficient Sapiom credits | Top up at app.sapiom.ai |
| 404 | Page not found or unreachable | Check URL is publicly accessible; try adding `wait` for slow pages |
| 429 | Rate limited | Exponential backoff and retry |

---

## Pricing

Both operations are flat-rate — no per-pixel or per-byte billing.

| Operation | Price |
|-----------|-------|
| `/tools/fetch-webpage` | $0.01 flat |
| `/tools/screenshot` | $0.01 flat |

Full-page screenshots (`capture_full_height: true`) cost the same as viewport
captures. The flat rate applies regardless of page length or image dimensions.