---
name: sapiom-web-search
description: >
  Search the web using Sapiom's API via Linkup or You.com — no vendor accounts,
  no API key management, no billing setup required. Use when an agent needs
  real-time information from the web, needs to ground responses in live data,
  needs to extract structured data from search results, or needs to fetch and
  convert a URL to clean markdown. Do NOT use for information the agent already
  knows reliably — web search costs money per call.
license: MIT
metadata:
  version: "1.0.0"
---

# Web Search via Sapiom

Sapiom routes web search through two providers — **Linkup** and **You.com** —
behind a single SDK. No vendor accounts, no separate API keys.

## When to Apply This Skill

Use this skill when:
- An agent needs information beyond its training cutoff
- Building RAG pipelines that need live web grounding
- Extracting structured data from the web (competitor pricing, job listings, etc.)
- Fetching and reading the content of a specific URL as clean markdown
- Verifying facts or checking current status of something

Default to `standard` depth. Only use `deep` when the query genuinely requires
multi-step retrieval — it costs ~9x more.

## Provider Selection

| Provider | Best For | Output Modes |
|----------|----------|--------------|
| **Linkup** | Sourced answers, structured extraction, URL fetching | `searchResults`, `sourcedAnswer`, `structured` |
| **You.com** | Live crawling, freshness filtering, full-page content | Web results with optional live crawl |

**Rule of thumb:** Use Linkup when you need a clean answer with citations or
structured JSON output. Use You.com when you need the full content of pages,
freshness filters, or are building a RAG pipeline that needs long markdown chunks.

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
  agentName: "my-agent",        // optional but recommended for dashboard tracking
  serviceName: "web-search",    // optional but recommended for dashboard tracking
});
```

## Linkup

Base URL: `https://linkup.services.sapiom.ai`

### Output Mode 1 — Search Results (raw)

Use when you want a list of results to process yourself.

```typescript
const response = await fetch(
  "https://linkup.services.sapiom.ai/v1/search",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      q: "TypeScript best practices 2025",
      depth: "standard",
      outputType: "searchResults",
    }),
  }
);

const data = await response.json();
for (const result of data.results) {
  console.log(`${result.title} — ${result.url}`);
  console.log(result.snippet);
}
```

### Output Mode 2 — Sourced Answer (recommended for Q&A)

Use when the agent needs a direct answer it can cite. Linkup runs the search
and synthesizes an answer with source references.

```typescript
const response = await fetch(
  "https://linkup.services.sapiom.ai/v1/search",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      q: "What are the latest developments in AI agent frameworks?",
      depth: "standard",
      outputType: "sourcedAnswer",
    }),
  }
);

const data = await response.json();
console.log(data.answer);
console.log("Sources:", data.sources); // array of { title, url }
```

### Output Mode 3 — Structured (extract into JSON schema)

Use when you need to extract structured data from the web. Pass a JSON schema
and Linkup returns data that matches it.

```typescript
const response = await fetch(
  "https://linkup.services.sapiom.ai/v1/search",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      q: "Top AI infrastructure startups funded in 2025",
      depth: "deep",
      outputType: "structured",
      structuredOutputSchema: {
        type: "object",
        properties: {
          companies: {
            type: "array",
            items: {
              type: "object",
              properties: {
                name: { type: "string" },
                fundingAmount: { type: "string" },
                focus: { type: "string" },
              },
            },
          },
        },
      },
      includeSources: true,
    }),
  }
);

const data = await response.json();
console.log(data.companies);
```

### Fetch a URL as Markdown

Converts any webpage to clean markdown — useful for reading documentation,
articles, or any page the agent needs to reason over.

```typescript
const response = await fetch(
  "https://linkup.services.sapiom.ai/v1/fetch",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      url: "https://example.com/article",
      renderJs: false, // set true for JS-heavy pages (costs more)
    }),
  }
);

const data = await response.json();
console.log(data.markdown);
```

## You.com

Base URL: `https://you-com.services.sapiom.ai`

You.com uses a GET request for search (not POST like Linkup).

### Basic Search

```typescript
const params = new URLSearchParams({
  query: "Best practices for building AI agents",
  count: "10",
});

const response = await fetch(
  `https://you-com.services.sapiom.ai/v1/search?${params}`
);

const data = await response.json();
for (const result of data.results.web) {
  console.log(`${result.title} — ${result.url}`);
}
```

### Search with Live Crawling (full page content)

Live crawling fetches the full content of each result page, not just snippets.
Use this for RAG pipelines where you need long-form content.

```typescript
const params = new URLSearchParams({
  query: "Sapiom AI agent infrastructure",
  count: "5",
  livecrawl: "web",
  livecrawl_formats: "markdown",
});

const response = await fetch(
  `https://you-com.services.sapiom.ai/v1/search?${params}`
);

const data = await response.json();
for (const result of data.results.web) {
  console.log(result.title);
  console.log(result.content); // full page markdown
}
```

### Freshness Filtering

```typescript
const params = new URLSearchParams({
  query: "AI funding rounds",
  freshness: "week", // day | week | month | year
  count: "10",
});
```

### Fetch Specific URLs

Retrieve the content of specific URLs you already know:

```typescript
const response = await fetch(
  "https://you-com.services.sapiom.ai/v1/contents",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      urls: [
        "https://example.com/article1",
        "https://example.com/article2",
      ],
      formats: ["markdown"], // html | markdown | metadata
    }),
  }
);

const data = await response.json();
for (const content of data.contents) {
  console.log(content.markdown);
}
```

## Common Patterns

### Grounding an LLM response with live search

```typescript
const searchResponse = await fetch(
  "https://linkup.services.sapiom.ai/v1/search",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      q: userQuery,
      depth: "standard",
      outputType: "sourcedAnswer",
    }),
  }
);

const { answer, sources } = await searchResponse.json();

// Pass into your LLM
const prompt = `
Answer the user's question using the following web search result.
Always cite your sources.

Search result: ${answer}
Sources: ${sources.map((s: any) => s.url).join(", ")}

User question: ${userQuery}
`;
```

### Choosing depth based on query complexity

```typescript
function getDepth(query: string): "standard" | "deep" {
  // Use deep only for complex research questions
  const complexIndicators = ["compare", "analysis", "comprehensive", "overview of"];
  return complexIndicators.some((word) => query.toLowerCase().includes(word))
    ? "deep"
    : "standard";
}
```

## Error Handling

```typescript
try {
  const response = await fetch("https://linkup.services.sapiom.ai/v1/search", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ q: query, depth: "standard", outputType: "sourcedAnswer" }),
  });

  if (!response.ok) {
    const status = response.status;
    if (status === 402) {
      // Insufficient Sapiom credits — top up at app.sapiom.ai or alert user
      throw new Error("Sapiom credits exhausted");
    }
    if (status === 429) {
      // Rate limited — back off and retry
      await new Promise((r) => setTimeout(r, 2000));
      // retry logic here
    }
    if (status === 400) {
      // Invalid parameters — check query and outputType
      throw new Error(`Bad request: ${await response.text()}`);
    }
  }

  return await response.json();
} catch (err) {
  // If search fails, fall back to model knowledge and flag as unverified
  console.error("Search failed, falling back to model knowledge:", err);
  return null;
}
```

## Cost & Pricing

| Provider | Operation | Price |
|----------|-----------|-------|
| Linkup | Standard search | $0.006/search |
| Linkup | Deep search | $0.055/search |
| Linkup | URL fetch (no JS) | $0.001/fetch |
| Linkup | URL fetch (with JS) | $0.006/fetch |
| You.com | 1–50 results | $0.006/search |
| You.com | 51–100 results | $0.008/search |
| You.com | Contents (1–10 URLs) | $0.01/request |

Check pricing estimates before running expensive searches using the free
price endpoints: `POST /v1/search/price` (Linkup) or `POST /v1/search/price` (You.com).

## Full API Reference

→ [references/search-api.md](references/search-api.md)