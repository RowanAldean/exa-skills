---
name: exa-crm-prospector
description: Generate, enrich, and compile high-signal prospect lists ready for CRM injection. Uses Exa Deep Search and sub-agents to build deduplicated CSV/JSON files perfectly formatted for tools like twenty-cli, Salesforce, or Hubspot without bloating the main context.
---

# Exa CRM Prospector Workflow

Use this skill when tasked with generating large, enriched prospect or lead lists (e.g., 50+ companies) to be injected into a CRM like Twenty, HubSpot, or Salesforce. 

This is a **Workflow Skill** designed to orchestrate high-volume data extraction while strictly protecting your context window. It bridges the gap between raw web search and clean CRM data entry.

## Architecture: Sub-Agent Driven Lead Gen

Do not run massive lead generation searches in the main context. For 100 leads, the raw JSON can exceed 100K tokens.

**The Orchestration Pipeline:**
1. **ICP & Schema Alignment (Main Agent)**: Map the user's CRM fields to an Exa `outputSchema`.
2. **Micro-Vertical Expansion (Main Agent)**: Break the core ICP into 5-10 specific queries.
3. **Delegation (Teammates)**: Spawn teammates to run Exa searches in parallel. **Crucially, teammates save their JSON to `/tmp/` files and NEVER print the raw data to the chat.**
4. **Python Compilation (Main Agent)**: Run a Python script to read the `/tmp/` files, deduplicate, and output pristine `crm_import_ready.csv` and `crm_import_ready.json` files.
5. **CRM Handoff**: Pass the compiled file to the user or to a CLI tool (e.g., `twenty-cli`) for injection.

---

## Step 1: Schema Design & ICP Alignment

First, determine the fields the target CRM requires. Design an `outputSchema` tailored to those exact fields.

**Critical Schema Constraints:**
- **Max 10 total properties** (including wrapper arrays/objects).
- **Enforce Brevity**: Every string description MUST contain a length limit (e.g., `"description": "Company overview in 12 words or less"`). This prevents massive text dumps and keeps CRM fields clean.
- Ensure you ask for `website_url` as it is the best unique identifier for deduplication.

## Step 2: Micro-Vertical Expansion

Instead of running one search for "Healthcare AI startups" with `num_results: 100` (which degrades quality), generate 5-10 highly specific "micro-verticals" to run in parallel.
- *Bad*: "Healthcare AI"
- *Good*: "Clinical trial patient matching platforms", "Medical imaging diagnostic AI startups", "EHR data analytics platforms"

Target ~20-30 results per micro-vertical.

## Step 3: Launch Teammates (Token Isolation)

Spawn teammates (sub-agents) to execute the searches. Give them explicit instructions:

> "You are a Prospecting Sub-Agent. Run an Exa search with `type: "deep"` using the following `query` and `outputSchema`. 
> **DO NOT output the results in your message.** 
> Save the raw JSON response directly to `/tmp/exa_leads_batch_[ID].json` and report back ONLY the number of companies found and the file path."

If the user wants 100 leads, spawn 2-3 teammates, assign each 2 micro-verticals, and let them work in parallel.

## Step 4: Python Compilation Script

Once all teammates report success, write and execute a Python script to compile the results. The script must:
1. Load all `/tmp/exa_leads_batch_*.json` files.
2. Extract the nested arrays of companies.
3. **Deduplicate** based on the company name or `website_url` (strip `http://`, `www.`, and trailing slashes for safety).
4. Export the data to both `crm_import_ready.csv` and `crm_import_ready.json`.

*Example Python Script Structure:*
```python
import json, csv, glob
from urllib.parse import urlparse

companies = []
for f in glob.glob("/tmp/exa_leads_batch_*.json"):
    try:
        data = json.load(open(f))
        # Adjust parsing based on the exact structure returned by Exa
        companies.extend(data.get("companies", []))
    except Exception as e:
        print(f"Skipping {f}: {e}")

seen_domains = set()
deduped = []

for c in companies:
    # Normalize domain for deduplication
    url = c.get("website_url", "")
    domain = urlparse(url).netloc.replace("www.", "") if url else c.get("company_name", "")
    
    if domain and domain not in seen_domains:
        seen_domains.add(domain)
        deduped.append(c)

# Output for JSON-based CLIs (like twenty-cli)
with open("crm_import_ready.json", "w") as f:
    json.dump(deduped, f, indent=2)

# Output for CSV bulk imports
if deduped:
    keys = deduped[0].keys()
    with open("crm_import_ready.csv", "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=keys)
        writer.writeheader()
        writer.writerows(deduped)

print(f"Successfully compiled and deduplicated {len(deduped)} CRM-ready records.")
```

## Step 5: CRM Injection

Once `crm_import_ready.json` and `crm_import_ready.csv` are generated, inform the user. 

If the user has provided a CLI tool (e.g., `twenty-cli`, `sfdx`), you can now safely loop through the compiled file and inject the records into their CRM system systematically.