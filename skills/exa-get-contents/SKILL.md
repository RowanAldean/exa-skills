# Exa Get Contents (Token-Efficient)

Use this skill to extract clean, LLM-ready content from known URLs using the Exa API via `curl`, while **strictly minimizing token usage** to keep context sizes manageable. 

**Note**: This is for when you *already have* the URLs. If you need to search the web to find URLs first, use the `exa-search` skill instead.

## Extraction Approaches

Choose the extraction mode that best balances your need for information against token cost. **Do NOT wrap extraction options in a `contents` object** (unlike the `/search` endpoint).

### 1. Highlights Extraction (Recommended)
Extracts key excerpts relevant to a query. This uses 10x fewer tokens than full text.

```bash
curl -X POST 'https://api.exa.ai/contents' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "urls": ["https://example.com/article", "https://example.com/another-post"],
  "highlights": {
    "query": "key findings and methodology",
    "maxCharacters": 4000
  }
}'
```

### 2. Structured Summary
Uses an LLM to generate an abstract or structured JSON. Excellent for analyzing dense company pages or long documents without reading the entire text.

```bash
curl -X POST 'https://api.exa.ai/contents' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "urls": ["https://example.com"],
  "summary": {
    "query": "Extract company overview and recent news",
    "schema": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "industry": { "type": "string" },
        "recent_news": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["name", "industry"]
    }
  }
}'
```

### 3. Subpage Crawling (Documentation/Deep Dives)
Automatically discover and extract content from linked subpages (e.g., API docs). Use `subpageTarget` to focus the crawl.

```bash
curl -X POST 'https://api.exa.ai/contents' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "urls": ["https://docs.example.com"],
  "subpages": 5,
  "subpageTarget": ["api", "reference", "guide"],
  "highlights": { "maxCharacters": 4000 }
}'
```

## Gotchas & Error Handling

- **Top-Level Keys**: `text`, `highlights`, and `summary` MUST be placed at the top level of the JSON payload. Do NOT nest them inside a `contents` object (that is only for the `/search` endpoint).
- **Silent Failures**: The API returns HTTP 200 even if individual URLs fail to scrape. You MUST check the `statuses` array in the JSON response to verify success.
  - E.g., check that `statuses[].status` is `"success"` and not `"error"`. Look for tags like `CRAWL_NOT_FOUND` or `CRAWL_LIVECRAWL_TIMEOUT` if an error occurs.
- **Freshness**: To bypass cache and force a live crawl, add `"maxAgeHours": 0` and a timeout `"livecrawlTimeout": 15000` to the top level of the JSON. Do not use the deprecated `livecrawl` parameter.
- **Deprecated Parameters**: Never use `useAutoprompt`, `stream`, `tokensNum`, `numSentences`, or `highlightsPerUrl`. 
- **Token warnings**: Avoid `text: true` without limits. If you must use text, always cap it: `"text": {"maxCharacters": 8000}`.