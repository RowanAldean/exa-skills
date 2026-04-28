---
name: exa-code-search
description: Search for code snippets, API usage, error resolutions, and GitHub implementations using the Exa API. Adapted from Exa MCP search patterns.
---

# Exa Code Search

Use this skill to perform targeted searches for code snippets, official documentation, debugging context, and GitHub implementations.

## Query Patterns: Code and Documentation

**Golden Rule**: Always include the programming language and framework/library in your query.

### 1. API Usage
Search for specific API implementation examples.

```bash
curl -X POST 'https://api.exa.ai/search' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "Stripe API create subscription Node.js code example",
  "type": "auto",
  "num_results": 5,
  "contents": {
    "highlights": { "max_characters": 4000 }
  }
}'
```

### 2. Error Resolution
Search for explanations and fixes for specific errors.

```bash
curl -X POST 'https://api.exa.ai/search' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "React hydration mismatch server client explanation fix",
  "type": "auto",
  "num_results": 10,
  "contents": {
    "highlights": { "max_characters": 4000 }
  }
}'
```

### 3. GitHub Implementations (Exa Code)
Use the `github` category to find open source code, repositories, and implementations directly on GitHub.

```bash
curl -X POST 'https://api.exa.ai/search' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "example project open source using [library]",
  "category": "github",
  "num_results": 10,
  "contents": {
    "highlights": { "max_characters": 4000 }
  }
}'
```

### 4. Fetch Known Docs
Use the `/contents` endpoint to read official docs when you already know the URL, extracting highlights to save context size.

```bash
curl -X POST 'https://api.exa.ai/contents' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "urls": ["https://docs.stripe.com/api/subscriptions"],
  "highlights": { "max_characters": 4000 }
}'
```
