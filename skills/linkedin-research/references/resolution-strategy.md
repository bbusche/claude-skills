# URL Resolution — Edge Cases and Detailed Strategy

Reference material for handling non-obvious resolution decisions during the URL processing step.
Read this when encountering edge cases not covered by the main SKILL.md workflow.

---

## LinkedIn Interstitial Pages

LinkedIn sometimes routes shortened URLs through an interstitial page before the destination:

- **`linkedin.com/safety/go/...`** — A safety check page. The actual destination URL is
  usually embedded in the query parameters or page content as a `data-url` attribute.
- **`linkedin.com/redir/redirect`** — An older redirect format. Extract the `url` query
  parameter for the destination.
- **Login walls** — If Playwright lands on `linkedin.com/login`, the resolution requires
  an authenticated session. Fall through to Tier 2 immediately.

When Playwright lands on any `linkedin.com` page that isn't the final destination:

1. Check query parameters for `url`, `redirect`, or `dest` keys
2. Check page content for external URLs in links or meta tags
3. If neither yields a result, mark as Tier 1 failure and proceed to web search

---

## URLs That Resolve to LinkedIn Content

Some `lnkd.in` URLs legitimately point to LinkedIn content:

- **LinkedIn articles** (`linkedin.com/pulse/...`) — These are valid destinations. NotebookLM
  can usually index these if they are public.
- **LinkedIn posts** (`linkedin.com/posts/...`) — These are valid but NotebookLM often
  cannot index them due to auth requirements. Flag in the report.
- **LinkedIn newsletters** (`linkedin.com/newsletters/...`) — Similar to articles, usually
  indexable if public.

When the resolved URL is on `linkedin.com`, verify it's a content page (not a login wall
or interstitial) before accepting it as resolved.

---

## Context Extraction Heuristics

When falling back to web search, the quality of the search query determines success.
Extract context using these heuristics:

### Numbered or bulleted lists

If the user pasted a numbered list with link text:

```
3. Do Things That Don't Scale — https://lnkd.in/abc123
```

The text before the URL ("Do Things That Don't Scale") is likely the article title.
Use it as a quoted search query: `"Do Things That Don't Scale"`.

### Inline links with descriptions

```
Check out this great article on retention metrics (https://lnkd.in/abc123) by Elena Verna
```

Extract "retention metrics" and "Elena Verna" as search terms.

### No context available

If a URL appears with no surrounding text, try these approaches in order:

1. Expand the shortened URL with Playwright to get at least a page title
2. Search for the shortened URL itself — sometimes third-party sites index the destination
3. Ask the user for context

---

## Duplicate and Overlapping URLs

- **Same destination, different short URLs:** Deduplicate and keep only one entry
- **Same article, different domains:** If a URL resolves to a repost or mirror, prefer
  the original source (earlier publication date, canonical domain)
- **Paywall variants:** If the resolved URL is behind a paywall, check if a free version
  exists on the author's blog, a preprint server, or an archive

---

## NotebookLM Source Compatibility

Not all resolved URLs will work as NotebookLM sources:

| URL Type | NotebookLM Support | Notes |
|---|---|---|
| Blog posts / articles | Usually works | Best source type |
| YouTube videos | Works | Transcripts are indexed |
| PDF links | Works | Direct PDF URLs preferred |
| Twitter/X threads | Sometimes | May fail if thread is long or account is private |
| LinkedIn posts | Rarely works | Auth-gated; flag in report |
| Substack articles | Usually works | Some may be paywalled |
| Google Docs (public) | Works | Must be publicly shared |
| Paywalled content | Does not work | Note in failure report |

When a source type is known to be problematic, note this preemptively in the resolution
table rather than waiting for the add-source step to fail.

---

## Rate Limiting and Timeouts

- **Playwright resolution:** Allow up to 15 seconds per URL for page load and redirect
  completion. LinkedIn's redirects can be slow.
- **Web search:** Standard search API limits apply. Space searches by 1-2 seconds if
  making many sequential queries.
- **NotebookLM source addition:** Add sources one at a time with no parallelism. If a
  source times out, wait 5 seconds before attempting the next one.
- **Large batches (15+ URLs):** Process in groups of 5, providing progress updates between
  batches to keep the user informed.
