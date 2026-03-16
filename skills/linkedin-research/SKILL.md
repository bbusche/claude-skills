---
name: linkedin-research
description: >
  Resolve LinkedIn shortened URLs (lnkd.in) to their final destination URLs and create a
  NotebookLM notebook with all resolved sources. Activates automatically whenever the user
  pastes or references more than one LinkedIn-style shortened URL (https://lnkd.in/...).
  Also activates on explicit /linkedin-research or intent like "resolve these LinkedIn links",
  "create a notebook from these LinkedIn URLs", or "what do these LinkedIn links point to".
---

# LinkedIn Research to NotebookLM

Resolve LinkedIn shortened URLs (`lnkd.in`) to their final destination URLs, then create a
NotebookLM notebook and add all resolved URLs as sources for AI-powered analysis.

---

## Purpose

LinkedIn posts frequently contain shortened `lnkd.in` URLs that are difficult to follow
programmatically. LinkedIn's URL shortener uses JavaScript-based redirects and often requires
an authenticated browser session, making standard HTTP redirect-following tools (curl, Python
requests, urllib) ineffective.

This skill solves the problem with a two-tier resolution strategy — first attempting
browser-based resolution via Playwright, then falling back to web search using contextual
clues from the surrounding post text. Once all URLs are resolved, the skill creates a
NotebookLM notebook and loads the destination content for AI-powered research.

---

## Prerequisites

Two tools must be installed and available on the system:

### notebooklm-py

A command-line tool for programmatic access to Google NotebookLM.

- **Install:** `pip install notebooklm-py`
- **Post-install:** `notebooklm skill install` (installs the NotebookLM Claude skill)
- **Authenticate:** `notebooklm login` (one-time Google OAuth via browser)
- **Verify:** `notebooklm status` (should show "Authenticated as: ...")

### Playwright (optional but recommended)

Used as the first-attempt URL resolver. Can be provided via:

- **Playwright MCP server** in Claude Desktop: `npx @playwright/mcp@latest --headless`
- **Python Playwright** for CLI usage: `pip install playwright && playwright install chromium`

If Playwright is not available, the skill falls back entirely to web-search-based resolution.

---

## When This Skill Activates

**Automatic activation:**
- The user's message contains **two or more** `lnkd.in` URLs (e.g., `https://lnkd.in/abc123`)

**Explicit invocation:**
- User says `/linkedin-research`

**Intent-based activation:**
- "Resolve these LinkedIn links"
- "Create a notebook from these LinkedIn URLs"
- "What do these LinkedIn links point to?"
- "Turn these LinkedIn links into a research notebook"

**Follow-up activation:**
- "Add these LinkedIn links to the notebook too"
- "Here are more links from LinkedIn"

---

## Workflow

### Step 1 — Extract URLs

Parse all `lnkd.in` URLs from the user's message. Also extract any surrounding context
(link text, numbered labels, topic descriptions) that can help identify the destination
article if URL resolution fails.

Build a working list:

| # | Shortened URL | Context (if any) |
|---|--------------|-------------------|

### Step 2 — Resolve URLs

For each shortened URL, attempt resolution using a **two-tier strategy**:

#### Tier 1: Playwright (browser-based redirect following)

Use Playwright to navigate to the shortened URL and capture the final URL after all
redirects complete:

1. Navigate to the `lnkd.in` URL in a headless browser
2. Wait for the page to finish loading (wait for network idle or a reasonable timeout)
3. Capture the final URL from the browser's address bar
4. If the final URL is still on `linkedin.com/safety/go/...` or a LinkedIn interstitial,
   extract the destination URL from the page content or query parameters

**Success criteria:** The final URL is NOT on `lnkd.in` or `linkedin.com` (unless the
destination genuinely is a LinkedIn page like a LinkedIn article or post).

**When Playwright fails:** LinkedIn often requires an authenticated session for redirect
resolution. If Playwright returns a LinkedIn login page, auth wall, or the URL doesn't
resolve, fall through to Tier 2.

#### Tier 2: Web search (context-based discovery)

When Playwright cannot resolve a URL, use the surrounding context to find the destination:

1. Extract any visible title, description, or topic from the user's message near the URL
2. Search the web for the article using that context as the query
3. Match results against the expected content type and topic
4. Present the best match as the resolved URL

If both tiers fail for a URL and no context is available, flag it for manual resolution
and continue with the remaining URLs.

### Step 3 — Present Results for Confirmation

Show the user a resolution table:

| # | Original URL | Resolved URL | Title | Resolution Method |
|---|-------------|-------------|-------|-------------------|

- **Resolution Method** values: `Playwright`, `Web Search`, or `Failed`
- For failed URLs, note the reason and ask the user if they can provide the title or topic

**Wait for user confirmation before proceeding to Step 4.** The user may want to correct
URLs, remove entries, or add context for failed resolutions.

### Step 4 — Create NotebookLM Notebook

Prompt the user for a notebook name, or suggest one based on the overall topic of the
resolved URLs:

```bash
notebooklm create "<NOTEBOOK_NAME>" --json
```

Parse the notebook ID from the JSON response. Then set context:

```bash
notebooklm use <notebook_id>
```

### Step 5 — Add Sources

For each successfully resolved URL, add it as a source:

```bash
notebooklm source add "<RESOLVED_URL>" --json
```

- Add sources sequentially (not in parallel) to avoid rate limiting
- Collect the `source_id` from each JSON response
- If a source fails to add, log the failure and continue with remaining URLs
- LinkedIn post URLs (`linkedin.com/posts/...`) will likely fail — note this in the report
- Do not retry failed sources unless the error suggests a transient issue

### Step 6 — Report Results

After all sources are added, present a summary:

**Notebook details:**
- Notebook name and ID
- Total sources added (success count / total attempted)

**Source table:**

| # | Title | Source ID | Status |
|---|-------|-----------|--------|

**Failed sources (if any):**

| # | URL | Reason |
|---|-----|--------|

**Next steps reminder — inform the user they can now:**
- `notebooklm source list` — check source processing status
- `notebooklm ask "question"` — chat with the content
- `notebooklm generate audio` — create a podcast summary
- `notebooklm generate report --format briefing-doc` — create a written briefing
- `notebooklm generate video` — create a video explainer
- `notebooklm generate quiz` — create a quiz from the content

---

## Advanced Features

### Adding to an Existing Notebook

If the user wants to add more LinkedIn links to a previously created notebook:

1. Set context to the existing notebook: `notebooklm use <notebook_id>`
2. Skip Step 4 (notebook creation)
3. Proceed with Steps 1-3 (extract and resolve) and Steps 5-6 (add and report)

### Mixed URL Types

If the user's message contains both `lnkd.in` URLs and regular URLs:

- Resolve only the `lnkd.in` URLs
- Pass regular URLs through as-is
- Include all URLs in the final notebook

### Batch Processing

For large lists (15+ URLs), process in batches of 5 and provide progress updates:

- "Resolving batch 1/3 (URLs 1-5)..."
- "Resolving batch 2/3 (URLs 6-10)..."

---

## Error Handling

| Error | Cause | Action |
|---|---|---|
| No `lnkd.in` URLs found | User pasted non-LinkedIn URLs | Inform the user; suggest other skills for different URL types |
| Playwright not available | MCP server not configured or Python Playwright missing | Skip to Tier 2 (web search) for all URLs |
| All URLs fail resolution | No context available and Playwright auth-blocked | Ask the user to provide titles or descriptions for each link |
| `notebooklm` not found | Not installed | Provide installation commands (see Prerequisites) |
| `notebooklm` auth failure | Session expired | Prompt user to run `notebooklm login` |
| Source fails to add | URL behind paywall or auth-gated | Log failure, continue with remaining sources |
| All sources fail | Auth issue or service outage | Run `notebooklm auth check` and report status |

---

## Why LinkedIn URLs Are Hard to Resolve

LinkedIn's `lnkd.in` URL shortener is intentionally resistant to programmatic resolution:

1. **JavaScript redirects** — The shortener serves an HTML page with a JS-based redirect
   rather than a standard HTTP 301/302 redirect. Tools like `curl -L` only follow HTTP
   redirects, so they stop at the JS page.

2. **Authentication gates** — LinkedIn frequently interposes a login wall between the
   shortened URL and the destination, even for links to external (non-LinkedIn) content.

3. **Interstitial pages** — Some links route through `linkedin.com/safety/go/...` which
   adds another JS-based hop before reaching the final destination.

4. **Session cookies** — Resolution often requires valid LinkedIn session cookies, meaning
   even a headless browser without an active LinkedIn login may fail.

This is why the skill uses web search as a robust fallback — searching for the article by
its title or topic almost always surfaces the canonical URL, bypassing LinkedIn's redirect
chain entirely.

---

## Autonomy Rules

**Run automatically (no confirmation needed):**
- Extracting URLs from the user's message
- Resolving URLs via Playwright and web search
- Creating the NotebookLM notebook
- Adding sources to the notebook

**Ask before proceeding:**
- Confirm the resolved URL list before creating the notebook and adding sources
- The user may want to correct misresolved URLs or provide context for failed ones

---

## Output Format

Use concise, scannable formatting throughout:

- **Tables** for URL lists, resolution results, and source summaries
- **Bold** for article titles and resolution methods
- **Code blocks** for any commands shown to the user
- Keep status updates brief:
  - "Resolving 14 LinkedIn shortened URLs..."
  - "Resolved 12/14 — 2 require manual context"
  - "Adding source 3/12: **Article Title**..."
  - "All 12 sources added successfully."

Do not repeat the full workflow description to the user — just execute it and report results.
