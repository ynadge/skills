# Search API Reference

Full parameter reference for Sapiom's web search capability.

---

## Linkup

**Base URL:** `https://linkup.services.sapiom.ai`

### POST /v1/search

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | Yes | Search query |
| `depth` | string | Yes | `standard` or `deep` |
| `outputType` | string | No | `searchResults` (default), `sourcedAnswer`, or `structured` |
| `maxResults` | number | No | Max results to return (1–100) |
| `includeImages` | boolean | No | Include images in results |
| `includeDomains` | string[] | No | Only include results from these domains |
| `excludeDomains` | string[] | No | Exclude results from these domains |
| `fromDate` | date | No | Only include results from this date onward |
| `toDate` | date | No | Only include results up to this date |
| `structuredOutputSchema` | object | Required if `outputType: "structured"` | JSON Schema object defining the output shape |
| `includeSources` | boolean | No | Include source citations with `structured` output |

#### Response — `searchResults`

```json
{
  "results": [
    {
      "title": "string",
      "url": "string",
      "snippet": "string"
    }
  ]
}
```

#### Response — `sourcedAnswer`

```json
{
  "answer": "string",
  "sources": [
    {
      "title": "string",
      "url": "string"
    }
  ]
}
```

#### Response — `structured`

```json
{
  // Shape matches your structuredOutputSchema
  "sources": [ { "title": "string", "url": "string" } ]  // if includeSources: true
}
```

---

### POST /v1/search/price

Free endpoint — returns cost estimate before running the search. Same parameters as `/v1/search`.

#### Response

```json
{
  "price": 0.006,
  "currency": "USD"
}
```

---

### POST /v1/fetch

Fetches a URL and returns its content as clean markdown.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | URL to fetch |
| `renderJs` | boolean | No | Render JavaScript before extraction (default: `false`) |
| `includeRawHtml` | boolean | No | Include raw HTML alongside markdown (default: `false`) |
| `extractImages` | boolean | No | Extract image URLs from the page (default: `false`) |

#### Response

```json
{
  "markdown": "string",
  "rawHtml": "string",   // if includeRawHtml: true
  "images": ["string"]   // if extractImages: true
}
```

---

### POST /v1/fetch/price

Free endpoint — returns cost estimate for a fetch operation.

---

### Linkup Pricing

| Operation | Price |
|-----------|-------|
| `depth: standard` | $0.006 |
| `depth: deep` | $0.055 |
| URL fetch (no JS) | $0.001 |
| URL fetch (`renderJs: true`) | $0.006 |

---

## You.com

**Base URL:** `https://you-com.services.sapiom.ai`

### GET /v1/search

Note: You.com search uses GET with query parameters, not a POST body.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Search query |
| `count` | number | No | Results count (1–100, default: 10) |
| `freshness` | string | No | `day`, `week`, `month`, or `year` |
| `country` | string | No | ISO 3166-2 country code (e.g. `US`, `GB`) |
| `language` | string | No | BCP 47 language code (e.g. `en`, `fr`) |
| `offset` | number | No | Pagination offset (0–9) |
| `safesearch` | string | No | `off`, `moderate`, or `strict` |
| `livecrawl` | string | No | `web`, `news`, or `all` — fetches full page content |
| `livecrawl_formats` | string | No | `html` or `markdown` (used with `livecrawl`) |

#### Response

```json
{
  "results": {
    "web": [
      {
        "title": "string",
        "url": "string",
        "snippet": "string",
        "content": "string"  // populated when livecrawl is set
      }
    ]
  }
}
```

---

### POST /v1/search/price

Free endpoint — returns cost estimate.

---

### POST /v1/contents

Fetch content from specific URLs you already know.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `urls` | string[] | Yes | URLs to fetch (1–10) |
| `formats` | string[] | No | `html`, `markdown`, or `metadata` (default: `["markdown"]`) |
| `crawl_timeout` | number | No | Timeout in seconds per URL, 1–60 (default: 10) |

#### Response

```json
{
  "contents": [
    {
      "url": "string",
      "markdown": "string",
      "html": "string",
      "metadata": {}
    }
  ]
}
```

---

### POST /v1/contents/price

Free endpoint — returns cost estimate.

---

### You.com Pricing

| Operation | Price |
|-----------|-------|
| Search 1–50 results | $0.006 |
| Search 51–100 results | $0.008 |
| Contents (1–10 URLs) | $0.01/request |

---

## Error Codes

| Code | Meaning | Action |
|------|---------|--------|
| 400 | Invalid request parameters | Check required fields and types |
| 402 | Payment required / insufficient credits | Top up at app.sapiom.ai |
| 429 | Rate limit exceeded | Back off and retry with exponential delay |