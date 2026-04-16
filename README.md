# Exa Skills
Skills for teaching agents how to use Exa's API and platform.

> [!CAUTION]
> This skills directory is not an official extension from the [Exa](https://exa.ai/) team. It is not maintained, affiliated with or contributed to by them and it's use comes at your own discretion.

## Commands
Commands are user-invocable slash commands that you explicitly call.

| Command | Description |
|---------|-------------|
| `/exa:crm-prospector` | Orchestrate high-volume lead generation and compile deduplicated JSON/CSV files ready for CRM injection |
| `/exa:market-research` | Run a comprehensive market research and competitor analysis workflow |
| `/exa:search` | Perform a token-efficient web search, company lookup, or people discovery |
| `/exa:get-contents` | Extract clean, LLM-ready content, summaries, or highlights from known URLs |
| `/exa:websets` | Run large-scale asynchronous searches with verification and data enrichment |

## Skills
Skills are contextual and auto-loaded based on your conversation. When a request matches a skill's triggers, the agent loads and applies the relevant skill to provide accurate, up-to-date guidance.

| Skill | Useful for |
|-------|------------|
| `exa-crm-prospector` | Orchestrating high-volume lead generation using sub-agents to fetch and deduplicate data perfectly mapped for CRM CLIs (like twenty-cli) |
| `exa-market-research` | Conducting comprehensive market research, competitor analysis, and building company/people lists using a multi-agent workflow |
| `exa-search` | Performing general web searches, company lookups, and people discovery while strictly minimizing token usage |
| `exa-get-contents` | Extracting clean content, summaries, or highlights from known URLs without searching the web |
| `exa-websets` | Performing large-scale asynchronous searches with verification (`criteria`) and structured data extraction (`enrichments`) |

## Installing

These skills work with any agent that supports the [Agent Skills](https://agentskills.io/home) standard, including Claude Code, OpenCode, OpenAI Codex, and Pi.

### npx skills

Install using the [`npx skills`](https://skills.sh) CLI:

```
npx skills add https://github.com/RowanAldean/exa-skills
```

### Clone / Copy

Clone this repo and copy the skill folders into the appropriate directory for your agent:

| Agent | Skill Directory | Docs |
|-------|-----------------|------|
| Claude Code | `~/.claude/skills/` | [docs](https://code.claude.com/docs/en/skills) |
| Cursor | `~/.cursor/skills/` | [docs](https://cursor.com/docs/context/skills) |
| OpenCode | `~/.config/opencode/skills/` | [docs](https://opencode.ai/docs/skills/) |
| OpenAI Codex | `~/.codex/skills/` | [docs](https://developers.openai.com/codex/skills/) |
| Pi | `~/.pi/agent/skills/` | [docs](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent#skills) |
