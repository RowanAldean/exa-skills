---
name: exa-websets
description: Perform large-scale asynchronous searches with verification and structured data extraction using Exa Websets. Use when you need to search and filter multiple URLs with criteria, or extract specific data points asynchronously.
---

# Exa Websets (Async Search & Enrichment)

Use this skill to perform large-scale asynchronous searches that require verification (`criteria`) and structured data extraction (`enrichments`). 

Unlike the synchronous `/search` and `/contents` endpoints, Websets run in the background. You define what to look for, Exa verifies each result against your criteria, applies data extraction, and you poll for the completed items.

## Workflow & Examples

### 1. Create a Webset
Define your query, verification criteria, and the specific data points you want to extract (enrichments).

```bash
curl -X POST "https://api.exa.ai/websets/v0/websets/" \
  -H "accept: application/json" \
  -H "content-type: application/json" \
  -H "x-api-key: $EXA_API_KEY" \
  -d '{
    "search": {
      "query": "Top AI research labs focusing on large language models",
      "count": 5,
      "criteria": [
        {"description": "Organization conducts AI research"}
      ]
    },
    "enrichments": [
      {"description": "Find founding year", "format": "number"},
      {"description": "CEO or Director name", "format": "text"}
    ]
  }'
```
*Note the returned `id` (e.g., `ws_abc123`). The status will initially be `"running"`.*

### 2. Poll for Completion
Websets are asynchronous. Check the status until it becomes `"idle"`. You can append `?expand=items` to get up to 100 items immediately in the same response.

```bash
curl -X GET "https://api.exa.ai/websets/v0/websets/ws_abc123?expand=items" \
  -H "accept: application/json" \
  -H "x-api-key: $EXA_API_KEY"
```

### Python SDK Alternative (Recommended for polling)
If working in Python, the SDK handles the polling loop and pagination natively:

```python
from exa_py import Exa
exa = Exa("YOUR_API_KEY")

# 1. Create
webset = exa.websets.create(params={"search": {"query": "Top AI labs", "count": 5}})

# 2. Wait for completion (blocks until status is 'idle')
webset = exa.websets.wait_until_idle(webset.id)

# 3. Paginate items
cursor = None
while True:
    page = exa.websets.items.list(webset_id=webset.id, cursor=cursor)
    for item in page.data:
        print(item.properties.url)
    if not page.has_more:
        break
    cursor = page.next_cursor
```

## Token Isolation Strategy

When paginating over Webset results and parsing `item.properties`, the context window can quickly fill up. **Always spawn a sub-agent / teammate** to orchestrate the Webset polling and data extraction. The teammate can handle the raw JSON payloads and summarize the findings into a clean Markdown table for the main context.

## ⚠️ Critical Gotchas

- **Async Execution**: Do not expect items in the response when creating a Webset. You MUST poll the webset ID until its status is `"idle"`.
- **Item Data Nesting (Common Trap)**: Extracted fields are strictly nested under `properties`. 
  - Use `item.properties.url` (NOT `item.url`).
  - Use `item.properties.company.name` or `item.properties.person.name` (NOT `item.company.name`).
- **Enrichment Results are Arrays**: Extracted enrichment values are ALWAYS arrays of strings, even for numbers or single values (e.g., `item.enrichments[0].result` is `["2015"]`, or `null` if not found).
- **Mapping Enrichment IDs**: Enrichment results on items only contain an `enrichmentId`, not the human-readable description. To know which value is which, you must map `webset.enrichments[].id` to `webset.enrichments[].description` from the main Webset object.
- **Python SDK casing**: The Python SDK uses `snake_case` properties. Use `page.has_more` and `page.next_cursor` instead of the JSON API's `hasMore` and `nextCursor`. Use `item.properties` to access the nested data.
- **Idempotency**: Use `"externalId": "your-custom-id"` in the create payload to avoid duplicating expensive Webset runs. The API will return 409 if it already exists.