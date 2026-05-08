---
name: exa-events
description: Find upcoming events, hackathons, meetups, and conferences using the Exa API. Focuses searches on popular event platforms (e.g., Luma, Eventbrite, Meetup, Devpost) and extracting event organizers.
---

# Exa Event Discovery

Use this skill to perform targeted web searches for events, hackathons, and conferences using the Exa API via `curl`. It relies on domain targeting and date filtering to find current and relevant gatherings.

## Event Search Approach

Since events are often hosted on specific community and ticketing platforms, filtering by `includeDomains` produces the most accurate results.

### 1. Finding Events and Hackathons

This search targets major event platforms like Luma, Devpost, Meetup, and Eventbrite. Adjust the `query` to specify the topic and location.

```bash
curl -X POST 'https://api.exa.ai/search' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "upcoming AI hackathons and tech meetups in London",
  "type": "auto",
  "num_results": 15,
  "includeDomains": ["lu.ma", "meetup.com", "eventbrite.com", "partiful.com", "devpost.com"],
  "contents": {
    "highlights": { "max_characters": 4000 }
  }
}'
```

*Tip: You can use `startPublishedDate` (e.g., `"2024-01-01T00:00:00.000Z"`) to ensure you only get recently created event pages, filtering out years-old past events.*

### 2. Finding Event Organizers and Community Builders

While you can extract host details directly from the event page highlights above, you can also pivot to finding event organizers directly using the `people` category search. 

```bash
curl -X POST 'https://api.exa.ai/search' \
  -H "x-api-key: $EXA_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "hackathon organizer and tech community builder in London",
  "category": "people",
  "num_results": 10,
  "contents": {
    "highlights": { "max_characters": 4000 }
  }
}'
```
*Note: The `people` category search does not support date filters. Domain filters are also restricted to LinkedIn domains when searching for people.*

## Key Parameters & Platforms for Events

- **Targeted Domains** (`includeDomains`): Restricting searches to event sites eliminates irrelevant content. Common domains include:
  - `devpost.com` (Best for hackathons)
  - `lu.ma` (Tech meetups and social gatherings)
  - `eventbrite.com` (General events and conferences)
  - `meetup.com` (Community groups)
  - `partiful.com` (Social events)
  - `splashthat.com` (Corporate and branded events)
- **Time Sensitivity**: Add `"maxAgeHours": 720` (approx 1 month) to the root of the JSON payload if you want to bypass the cache and prioritize fresh pages.

## Token Isolation Strategy (Critical)

For heavy search operations that pull in large amounts of data (like multiple event descriptions):
1. **Never run Exa searches directly in the main context**. 
2. Spawn a sub-agent / teammate to act as a worker.
3. Have the teammate run the Exa search, read the highlights, and extract the relevant entities (Event Name, Date, URL, Organizer Name/Contact).
4. The teammate should return only a distilled output (e.g., a compact markdown table) to the main context.
