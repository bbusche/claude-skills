# Video Curation — Edge Cases and Detailed Criteria

Reference material for handling non-obvious curation decisions during the ranking step.
Read this when encountering edge cases not covered by the main SKILL.md criteria.

---

## Sponsored and Affiliate Content

Videos with heavy sponsorship or affiliate-driven content are not automatically excluded,
but they require additional scrutiny:

- **Acceptable:** Creator discloses sponsorship but provides genuine, balanced review
- **Downrank:** Video is primarily a product placement with minimal educational value
- **Exclude:** Video exists solely to sell a product/service with no substantive content

When in doubt, check whether the creator's recommendations align with the broader
consensus across non-sponsored videos on the same topic.

---

## Tutorial Series vs. Standalone Videos

When a creator has a multi-part series on the topic:

- **Prefer standalone videos** that cover the topic comprehensively in a single video
- **Include a series entry** only if it's the definitive resource and no standalone
  equivalent exists
- If including a series video, prefer Part 1 or an overview/summary episode
- Note in the "Why Selected" column that it's part of a series

---

## Live Streams and Podcasts

Live stream recordings and podcast episodes are valid sources but apply extra filtering:

- **Include:** Structured live streams with clear topic focus and expert guests
- **Downrank:** Unedited live streams with excessive off-topic chat or filler
- **Exclude:** Multi-hour general streams where the relevant topic is only a small segment

For podcast-format videos, prefer episodes dedicated to the topic over episodes that
briefly mention it alongside many other subjects.

---

## Non-English Content

By default, curate English-language content unless the user specifies otherwise.

- If the user requests content in a specific language, adjust the search query accordingly
- If a non-English video appears in results and is clearly the authoritative resource on
  the topic (e.g., original creator's native language), note this and ask the user if they
  want it included

---

## Duplicate and Reposted Content

- **Prefer originals:** If the same content appears on multiple channels, include only the
  original creator's upload (identifiable by earlier upload date and channel ownership)
- **Exclude mirrors:** Re-uploaded or clipped content from other channels without
  attribution
- **Handle updates:** If a creator has both an older and updated version of the same
  tutorial, prefer the newer version unless the older one is substantially more comprehensive

---

## View Count Interpretation

View counts should be interpreted relative to the topic's niche:

| Topic Type | "High Views" Threshold | "Low Views" Threshold |
|---|---|---|
| Mainstream tech (AI, programming) | >100K | <5K |
| Niche technical (specific framework/tool) | >10K | <500 |
| Emerging/new topics (<6 months old) | >5K | <200 |

A video with 2K views on a niche topic may be more authoritative than a 500K-view
video on a broad topic that only superficially covers the niche.

---

## Duration Guidelines

| Duration | Classification | Default Action |
|---|---|---|
| <60 seconds | Short/Reel | Exclude |
| 1-5 minutes | Brief overview | Include only if uniquely valuable |
| 5-20 minutes | Standard content | Preferred range |
| 20-45 minutes | Deep dive | Preferred for complex topics |
| 45-90 minutes | Full course/lecture | Include if comprehensive; note length |
| >90 minutes | Extended content | Include only if no shorter alternative exists |

---

## Channel Authority Signals

When evaluating channel quality, consider:

- **Strong signals:** Consistent upload schedule, focused topic area, professional
  production, community engagement (comments enabled with creator responses)
- **Neutral signals:** Mixed topic coverage, moderate subscriber count, occasional uploads
- **Weak signals:** Very new channel (<10 videos), no other content on the topic,
  disabled comments, suspiciously high view counts relative to subscriber count
