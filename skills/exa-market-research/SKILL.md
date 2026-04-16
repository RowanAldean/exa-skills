---
name: exa-market-research
description: Conduct comprehensive market research, competitor analysis, and build company lists. Use when researching a specific company, discovering startups in a sector, analyzing competitors, or generating lists of key people and news within an industry.
---

# Exa Market Research Workflow

Use this skill when tasked with high-level market research, competitor analysis, or building company/people lists. This is a **Workflow Skill** designed to help you think like a Research Coordinator. 

Do not write raw `curl` requests in the main context to prevent token bloat. Instead, orchestrate the research process by delegating to sub-agents.

## The Research Protocol

### 1. Token Isolation (Critical)
Never run Exa searches in the main context. The JSON responses are too large and will pollute the user's view.
- **Spawn a teammate** (sub-agent) for the duration of the research.
- Instruct the teammate to utilize the `exa-search` skill internally to gather the data.
- The teammate must process the results and return *only* the distilled output (e.g., a compact markdown list or table) to the main context.

### 2. Query Tuning & Expansion
Exa returns different results for different phrasings. Instruct your sub-agent to:
- Generate 2-3 query variations for broad coverage (e.g., "AI infrastructure startups", "companies building AI infrastructure", "AI infra platforms").
- Run them sequentially or concurrently and merge the results, deduplicating by URL.
- **Dynamic Sizing**: If the user asks for "a few", fetch 10-20. If "comprehensive", fetch 50-100. If ambiguous, ask before searching.

### 3. Progressive Deep Dives (Using Categories)
Instruct your sub-agents to use the proper `category` filters based on the phase of research:

- **Phase 1 (Discovery)**: Use `category: "company"` to find rich metadata like headcount, location, funding, and revenue.
- **Phase 2 (People & Talent)**: Use `category: "people"` to find LinkedIn profiles for founders or key executives (e.g., "VP Engineering at [Company]").
- **Phase 3 (News & Sentiment)**: Use `category: "news"` to find recent press coverage and announcements.
- **Phase 4 (Broad Context)**: Run without a category (`type: "auto"`) for general web context and technical deep dives.

### 4. ⚠️ 400-Error Crash Prevention
Ensure your sub-agent is aware of these strict API rules when using `exa-search`:
- If `category: "company"` is used, **DO NOT** use date filters (`startPublishedDate`, etc.) or domain filters (`includeDomains`, `excludeDomains`). Doing so will crash the search.
- If using `includeText` or `excludeText`, it must be a **single-item array** (e.g., `["AI"]`). Multiple items will crash the search across all categories.

## Desired Output Format
When your sub-agent reports back, ensure the final output presented to the user contains:
1. **Results**: A structured list or table (e.g., Company, Description, Location, Funding).
2. **Sources**: Links to the company homepages or articles (1-line relevance each).
3. **Notes**: Any conflicts found in the data or uncertainties.