---
name: yt-research
description: >
  Search YouTube for videos on a topic, rank and curate the best results, create a
  NotebookLM notebook, and add all curated videos as sources for AI-powered analysis.
  Use this skill whenever the user asks to research YouTube, find YouTube videos, search
  YouTube for content on a topic, or wants to build a curated video research collection.
  Trigger on any request containing phrases like: research YouTube, find YouTube videos,
  search YouTube for, YouTube videos about, video content on, or explicit /yt-research.
  Also trigger when the user expresses intent to collect video sources for a NotebookLM
  notebook or wants to create a research notebook from YouTube content. Always activate
  even for follow-up requests like "add more videos on this topic" or "search for
  additional videos about X."
---

# YouTube Research to NotebookLM

Search YouTube for video content on a topic, rank and curate the top results, create a
NotebookLM notebook, and add all curated videos as sources for further AI-powered
analysis including podcast generation, briefing docs, and interactive Q&A.

---

## Purpose

Enable rapid, structured research on any topic by leveraging YouTube's vast library of
expert content. Rather than manually watching dozens of videos, this skill curates the
best sources and loads them into Google NotebookLM where AI can synthesize, summarize,
and answer questions across all sources simultaneously.

The end-to-end workflow transforms a topic query into a fully populated research notebook
in under 5 minutes, ready for deep analysis.

---

## Prerequisites

Two command-line tools must be installed and available on the system:

### yt-dlp

A command-line tool for searching and downloading YouTube metadata.

- **Install (Mac):** `brew install yt-dlp` or `pip install yt-dlp`
- **Install (Windows):** `pip install yt-dlp` or download from [yt-dlp releases](https://github.com/yt-dlp/yt-dlp/releases)
- **Verify:** `yt-dlp --version`

### notebooklm-py

A command-line tool for programmatic access to Google NotebookLM.

- **Install:** `pip install notebooklm-py`
- **Post-install:** `notebooklm skill install` (installs the NotebookLM Claude skill)
- **Authenticate:** `notebooklm login` (one-time Google OAuth via browser)
- **Verify:** `notebooklm status` (should show "Authenticated as: ...")

If either tool is missing, inform the user and provide the installation commands above.

### Playwright (for notebooklm-py)

The `notebooklm-py` CLI requires Python Playwright with Chromium:

- **Install:** `pip install playwright && playwright install chromium`

---

## When This Skill Activates

**Explicit invocation:**
- User says `/yt-research <topic>`

**Intent-based activation:**
- "Research YouTube for videos about..."
- "Find YouTube videos about..."
- "Search YouTube for content on..."
- "I want to find video content about..."
- "Create a research notebook from YouTube videos on..."
- "What are the best YouTube videos about..."

**Follow-up activation:**
- "Add more videos on this topic"
- "Search for additional videos about X"
- "Can you also find videos about Y and add them to the notebook?"

---

## Workflow

### Step 1 — Search YouTube

Use `yt-dlp` to search YouTube for 15 results on the given topic:

```bash
yt-dlp "ytsearch15:<TOPIC>" --flat-playlist --print "%(id)s|%(title)s|%(channel)s|%(view_count)s|%(duration)s|%(upload_date)s|%(description).300s" 2>&1
```

This returns pipe-delimited fields:

| Field | Description |
|---|---|
| `id` | YouTube video ID |
| `title` | Video title |
| `channel` | Channel name |
| `view_count` | Total views |
| `duration` | Duration in seconds |
| `upload_date` | Upload date (YYYYMMDD or NA) |
| `description` | First 300 characters of description |

**Multiple searches:** If the topic is broad or has multiple facets, run 2-3 searches with
different keyword variations to maximize coverage. For example, a topic like "secure AI
assistant deployment" might warrant searches for both "OpenClaw secure setup" and
"AI agent VPS Mac Mini security."

**User-specified videos:** If the user provides specific YouTube URLs or channel URLs,
fetch those separately using `yt-dlp` with the direct URL and include them in the
candidate pool regardless of search ranking. These are always included in the final
curated list.

### Step 2 — Rank and Curate

From all search results, select the **top 5-8 most relevant and high-quality videos**
based on these criteria:

1. **Relevance** — Title and description must closely match the search topic. Tangential
   content is excluded even if popular.

2. **Quality signals** — Prefer established channels with consistent content, higher view
   counts, and longer-form content (over 5 minutes) that suggests depth and substance.

3. **Recency** — Prefer newer content when the topic benefits from it (e.g., software
   tutorials, current events). For evergreen topics, recency matters less.

4. **Diversity** — Include a mix of perspectives: tutorials, deep dives, critiques,
   discussions, comparisons, and practical walkthroughs. Avoid selecting multiple videos
   that cover the exact same ground.

5. **Exclude** — Shorts (under 60 seconds), obvious clickbait, tangentially related
   content, duplicate topics from the same channel, and low-effort compilation videos.

Present the curated list to the user in a table:

| # | Title | Channel | Views | Duration | Why Selected |
|---|-------|---------|-------|----------|--------------|

Also list excluded videos with brief reasons (e.g., "too short," "overlaps with #3,"
"tangential to topic").

**Wait for user confirmation before proceeding to Step 3.** The user may want to add,
remove, or swap videos before the notebook is created.

### Step 3 — Create NotebookLM Notebook

```bash
notebooklm create "YT Research: <TOPIC>" --json
```

Parse the notebook ID from the JSON response (`id` field). Then set context:

```bash
notebooklm use <notebook_id>
```

**Naming convention:** Always prefix with "YT Research: " followed by a concise topic
description. Keep the title under 80 characters.

### Step 4 — Add Videos as Sources

For each curated video, add it as a YouTube source:

```bash
notebooklm source add "https://www.youtube.com/watch?v=<VIDEO_ID>" --json
```

- Add sources sequentially (not in parallel) to avoid rate limiting
- Collect the `source_id` from each JSON response
- If a source fails to add, log the failure and continue with remaining videos
- Do not retry failed sources unless the error suggests a transient issue

### Step 5 — Report Results

After all sources are added, present a summary:

**Notebook details:**
- Notebook name and ID
- Total sources added (success count / total attempted)

**Source table:**

| # | Video | Source ID | Status |
|---|-------|-----------|--------|

**Next steps reminder — inform the user they can now:**
- `notebooklm source list` — check source processing status
- `notebooklm ask "question"` — chat with the video content
- `notebooklm generate audio` — create a podcast summary
- `notebooklm generate report --format briefing-doc` — create a written briefing
- `notebooklm generate video` — create a video explainer
- `notebooklm generate quiz` — create a quiz from the content

---

## Advanced Features

### Adding to an Existing Notebook

If the user wants to add more videos to a previously created notebook:

1. Set context to the existing notebook: `notebooklm use <notebook_id>`
2. Skip Step 3 (notebook creation)
3. Proceed with Steps 1-2 (search and curate) and Steps 4-5 (add and report)

### Channel-Specific Searches

If the user specifies a YouTube channel, fetch recent videos from that channel:

```bash
yt-dlp "https://www.youtube.com/@<CHANNEL_HANDLE>" --flat-playlist --print "%(id)s|%(title)s|%(view_count)s|%(duration)s|%(upload_date)s|%(description).200s" --playlist-end 20 2>&1
```

Include relevant videos from the channel in the candidate pool.

### Multi-Topic Research

If the user requests research across multiple related subtopics:

1. Run separate searches for each subtopic
2. Deduplicate results across searches
3. Present a single curated list organized by subtopic
4. Create one notebook with all sources (or separate notebooks if the user prefers)

---

## Error Handling

| Error | Cause | Action |
|---|---|---|
| `yt-dlp` returns no results | Query too specific or no matches | Suggest broader or alternative search terms |
| `yt-dlp` not found | Not installed | Provide installation command for user's platform |
| `notebooklm` auth failure | Session expired | Prompt user to run `notebooklm login` |
| Source fails to add | Processing error or rate limit | Log failure, continue with remaining videos |
| All sources fail | Auth issue or service outage | Run `notebooklm auth check` and report status |
| `notebooklm` not found | Not installed | Provide installation commands (see Prerequisites) |

---

## Autonomy Rules

**Run automatically (no confirmation needed):**
- YouTube search via yt-dlp (read-only, no side effects)
- Creating the NotebookLM notebook
- Adding sources to the notebook
- Checking source processing status

**Ask before proceeding:**
- Confirm the curated video list before creating the notebook and adding sources
- The user may want to adjust the selection, add specific videos, or change the topic focus

---

## Output Format

Use concise, scannable formatting throughout:

- **Tables** for search results, curated lists, and source summaries
- **Bold** for video titles and channel names in prose
- **Code blocks** for any commands shown to the user
- **Numbered steps** for the workflow progress

Keep status updates brief:
- "Searching YouTube for 15 results on '<topic>'..."
- "Adding source 3/8: **Video Title** (Channel)..."
- "All 8 sources added successfully."

Do not repeat the full workflow description to the user — just execute it and report results.
