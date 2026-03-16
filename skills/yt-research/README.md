# yt-research

A Claude skill that searches YouTube for the best videos on any topic, curates and ranks results, creates a Google NotebookLM notebook, and adds all curated videos as sources for AI-powered analysis.

## What It Does

When triggered, this skill guides Claude to:

1. **Search YouTube** — uses `yt-dlp` to find 15 candidate videos matching the topic, with support for multiple search queries and user-specified URLs/channels
2. **Rank and curate** — selects the top 5-8 videos based on relevance, quality signals (views, duration, channel authority), recency, and diversity of perspective
3. **Present for approval** — shows the curated list in a table with selection rationale, waits for user confirmation
4. **Create a NotebookLM notebook** — sets up a titled research notebook via the `notebooklm` CLI
5. **Add all videos as sources** — loads each curated video into the notebook for AI indexing
6. **Report results** — provides a summary with source IDs and next-step commands

## Triggers

This skill activates whenever the user asks to research YouTube content, including:

- Explicit: `/yt-research <topic>`
- Intent-based: "research YouTube for...", "find YouTube videos about...", "search YouTube for content on..."
- Follow-ups: "add more videos on this topic", "search for additional videos about X"

## File Structure

```
yt-research/
├── README.md                  ← this file (GitHub display)
├── SKILL.md                   ← skill instructions loaded by Claude
├── LICENSE.txt                ← MIT license
└── references/
    └── curation-criteria.md   ← detailed ranking criteria and edge cases
```

### SKILL.md

Contains the YAML frontmatter (`name`, `description`) that triggers the skill, followed by the full workflow: prerequisites, five-step process, advanced features, error handling, and autonomy rules.

### references/curation-criteria.md

Loaded on demand for edge cases in video curation — guidance on handling sponsored content, tutorial series, live streams, and non-English results.

## Prerequisites

| Tool | Purpose | Install |
|---|---|---|
| `yt-dlp` | YouTube search and metadata extraction | `brew install yt-dlp` or `pip install yt-dlp` |
| `notebooklm-py` | Google NotebookLM CLI access | `pip install notebooklm-py` |
| `playwright` | Browser automation (required by notebooklm-py) | `pip install playwright && playwright install chromium` |

After installing `notebooklm-py`, authenticate once:

```bash
notebooklm login          # Opens browser for Google OAuth
notebooklm status         # Verify authentication
```

## Curation Criteria

The skill evaluates candidate videos on five dimensions:

| Criterion | What It Measures |
|---|---|
| **Relevance** | Title and description match to the search topic |
| **Quality signals** | View count, channel authority, content depth (>5 min preferred) |
| **Recency** | Publication date relevance to the topic's currency |
| **Diversity** | Mix of tutorials, deep dives, critiques, and discussions |
| **Exclusions** | Shorts (<60s), clickbait, tangential content, same-channel duplicates |

## What You Can Do After

Once the notebook is populated, use NotebookLM to:

- `notebooklm ask "question"` — chat with all video content at once
- `notebooklm generate audio` — create a podcast summary
- `notebooklm generate report --format briefing-doc` — create a written briefing
- `notebooklm generate video` — create a video explainer
- `notebooklm generate quiz` — create a quiz from the content
- `notebooklm source list` — check source processing status

## Installation

Copy the `yt-research/` folder into your Claude skills directory (typically `~/.claude/skills/` or the path configured in your Claude settings), then restart Claude.

## Author

Built using the [Claude Skill framework](https://github.com/anthropics/skills).
