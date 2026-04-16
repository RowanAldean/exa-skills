---
name: exa-search
description: Perform web searches, company research, and people discovery using the Exa API. Use when you need to search the web, find people, or look up companies, and need to minimize token usage via targeted highlights.
---

# Exa Token-Efficient Search

Use this skill to perform web searches, company research, and people discovery using the Exa API via `curl`, while **strictly minimizing token usage** to keep context sizes manageable.

## Search Approach

Always use `type: "auto"` for general queries and extract token-efficient `highlights` instead of full `text`.

### 1. General Web Search

```bash
curl -X POST 'https://api.exa.ai/search' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "YOUR_SPECIFIC_QUERY",
  "type": "auto",
  "num_results": 10,
  "contents": {
    "highlights": { "max_characters": 4000 }
  }
}'
```

### 2. People Search
Find individuals by role, expertise, or what they work on.
- **Rule**: Use SINGULAR form (e.g., "software engineer", not "software engineers").
- **Rule**: No date or text filters are supported.

```bash
curl -X POST 'https://api.exa.ai/search' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "software engineer working on distributed systems",
  "category": "people",
  "num_results": 10,
  "contents": {
    "highlights": { "max_characters": 4000 }
  }
}'
```

### 3. Company Search
Find companies by industry or criteria.

```bash
curl -X POST 'https://api.exa.ai/search' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "AI startup in healthcare",
  "category": "company",
  "num_results": 10,
  "contents": {
    "highlights": { "max_characters": 4000 }
  }
}'
```

### 4. Fetch Known URLs
Use `/contents` to extract highlights when you already have specific URLs.

```bash
curl -X POST 'https://api.exa.ai/contents' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "urls": ["https://example.com/article"],
  "highlights": { "max_characters": 4000 }
}'
```

## Gotchas

- **Nesting of `highlights`**: On the `/search` endpoint, `highlights` MUST be nested inside `contents` (`"contents": {"highlights": {...}}`). On the `/contents` endpoint, `highlights` is a top-level key.
- **Token warnings**: Never use `"text": {"max_characters": ...}` unless full contiguous text is explicitly required, as it severely increases token cost. Default to `highlights`.
- **Deprecated parameters**: Never use `useAutoprompt`, `stream`, `tokensNum`, `includeUrls`, `excludeUrls`, `numSentences`, or `highlightsPerUrl`.
- **Domain filtering**: Use `includeDomains` or `excludeDomains` as arrays of strings. Note that for `category: "people"`, `includeDomains` ONLY accepts LinkedIn domains.
- **Freshness Control**: If you need real-time data or want to bypass the cache, add `"maxAgeHours": 0` to your JSON payload. Do not use the deprecated `livecrawl` parameter.
