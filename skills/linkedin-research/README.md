# linkedin-research

A Claude skill that resolves LinkedIn shortened URLs (`lnkd.in`) to their final destination URLs and creates a Google NotebookLM notebook with all resolved content as sources for AI-powered analysis.

## What It Does

When triggered, this skill guides Claude to:

1. **Extract URLs** — parses all `lnkd.in` shortened URLs from the user's message along with surrounding context
2. **Resolve URLs** — uses a two-tier strategy: Playwright browser automation first, web search fallback second
3. **Present for approval** — shows a resolution table with original URLs, resolved destinations, and resolution method
4. **Create a NotebookLM notebook** — sets up a titled research notebook via the `notebooklm` CLI
5. **Add all URLs as sources** — loads each resolved URL into the notebook for AI indexing
6. **Report results** — provides a summary with source IDs and next-step commands

## Triggers

This skill activates automatically whenever the user's message contains **two or more** `lnkd.in` URLs:

- Automatic: Detects `https://lnkd.in/...` patterns in user messages
- Explicit: `/linkedin-research`
- Intent-based: "resolve these LinkedIn links", "create a notebook from these LinkedIn URLs"

## Why This Exists

LinkedIn's `lnkd.in` URL shortener is notoriously difficult to resolve programmatically:

| Method | Works? | Why |
|---|---|---|
| `curl -L` | No | LinkedIn uses JavaScript redirects, not HTTP 301/302 |
| Python `requests` / `urllib` | No | Same reason — JS redirect not followed |
| Headless browser (no session) | Sometimes | LinkedIn often requires authentication cookies |
| Headless browser (with session) | Yes | Requires a logged-in LinkedIn session |
| Web search by article title | Yes | Bypasses LinkedIn entirely by finding the canonical URL |

This skill combines Playwright (for cases where browser-based resolution works) with web search (as a reliable fallback) to resolve URLs regardless of LinkedIn's redirect behavior.

## File Structure

```
linkedin-research/
├── README.md                  ← this file (GitHub display)
├── SKILL.md                   ← skill instructions loaded by Claude
├── LICENSE.txt                ← MIT license
└── references/
    └── resolution-strategy.md ← detailed resolution logic and edge cases
```

### SKILL.md

Contains the YAML frontmatter (`name`, `description`) that triggers the skill, followed by the full workflow: prerequisites, two-tier resolution strategy, NotebookLM integration, error handling, and autonomy rules.

### references/resolution-strategy.md

Loaded on demand for edge cases in URL resolution — guidance on handling LinkedIn interstitials, auth walls, mixed URL types, and batch processing.

## Prerequisites

| Tool | Purpose | Install |
|---|---|---|
| `notebooklm-py` | Google NotebookLM CLI access | `pip install notebooklm-py` |
| `playwright` | Browser automation (URL resolution + notebooklm-py dependency) | `pip install playwright && playwright install chromium` |

After installing `notebooklm-py`, install the Claude skill and authenticate:

```bash
notebooklm skill install  # Installs the NotebookLM Claude skill
notebooklm login          # Opens browser for Google OAuth
notebooklm status         # Verify authentication
```

For Claude Desktop users, the Playwright MCP server provides browser-based resolution:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--headless"]
    }
  }
}
```

## Resolution Strategy

The skill uses a two-tier approach for each shortened URL:

| Tier | Method | When It Works | When It Fails |
|---|---|---|---|
| **1. Playwright** | Navigate headless browser to URL, capture final address | URL resolves without LinkedIn auth | LinkedIn requires login or serves interstitial |
| **2. Web Search** | Search for article by title/context from surrounding text | User provided context or link text | No context available and title unknown |

## What You Can Do After

Once the notebook is populated, use NotebookLM to:

- `notebooklm ask "question"` — chat with all content at once
- `notebooklm generate audio` — create a podcast summary
- `notebooklm generate report --format briefing-doc` — create a written briefing
- `notebooklm generate video` — create a video explainer
- `notebooklm generate quiz` — create a quiz from the content
- `notebooklm source list` — check source processing status

## Installation

Copy the `linkedin-research/` folder into your Claude skills directory (typically `~/.claude/skills/` or the path configured in your Claude settings), then restart Claude.

## Author

Built using the [Claude Skill framework](https://github.com/anthropics/skills).
