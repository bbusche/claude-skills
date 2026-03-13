# supplement-evaluator

A Claude skill that delivers rigorous, scientifically honest evaluations of dietary supplements and nutraceutical ingredients — acting as a knowledgeable, neutral scientific reviewer rather than a marketer or a blanket skeptic.

## What It Does

When triggered, this skill guides Claude to:

1. **Search for evidence** — finds peer-reviewed RCTs, systematic reviews, and meta-analyses specifically for the named ingredient or brand
2. **Catalog the studies** — extracts design, population, dose, endpoints, results, and funding source for each trial
3. **Apply critical appraisal** — flags industry funding, small N, within-group-only significance, and other common sources of bias
4. **Contextualize the mechanism** — separates in vitro/animal mechanistic data from human clinical evidence
5. **Give practical guidance** — effective dose range, timing, population specificity, and stack considerations
6. **Deliver a calibrated verdict** — tiered language keyed to actual evidence quality (not marketing claims)

## Triggers

This skill activates whenever the user asks about any supplement, vitamin, mineral, herbal extract, branded ingredient, or nutraceutical, including:

- Named compounds: ashwagandha, theanine, berberine, creatine, collagen, bacopa, lion's mane, rhodiola, HMB, citrulline, phosphatidylserine, and more
- Branded ingredients: KSM-66, SunTheanine, AlphaWave, Sensoril, BerberVis, Kre-Alkalyn, and similar
- General questions about supplement stacks, dosing, cycling, or whether a product's claims are supported by evidence

## File Structure

```
supplement-evaluator/
├── README.md                  ← this file (GitHub display)
├── SKILL.md                   ← skill instructions loaded by Claude
└── references/
    └── category-notes.md      ← appraisal notes for specific supplement categories
```

### SKILL.md

Contains the YAML frontmatter (`name`, `description`) that triggers the skill, followed by the full evaluation methodology: core principles, six-step workflow, output format, and quality benchmark.

### references/category-notes.md

Loaded on demand when evaluating supplements in specific categories:

- Adaptogens (ashwagandha, rhodiola, eleuthero, schisandra)
- Testosterone-supporting ingredients
- Nootropics and cognitive enhancers
- Sleep and stress compounds
- Cardiovascular / lipid-modifying nutraceuticals
- Body composition and performance ingredients

## Evidence Standards

The skill enforces a strict evidence hierarchy:

| Evidence Level | Verdict Language |
|---|---|
| Multiple independent RCTs, consistent findings | "Evidence supports efficacy for [X]" |
| Industry-funded RCTs only, no independent replication | "Possibly effective; independent replication needed" |
| Within-group significance only, no placebo separation | "Suggestive but inconclusive; placebo effect cannot be ruled out" |
| Animal / in vitro only | "Mechanistically plausible; no adequate human evidence" |
| No credible evidence | "Not supported by published human evidence" |

## What This Skill Does NOT Do

- Provide personalized medical advice or drug interaction guidance
- Make dosing recommendations for individuals with medical conditions
- Evaluate prescription pharmaceuticals or controlled substances
- Assess supplement quality, purity, or third-party testing certifications

## Installation

Copy the `supplement-evaluator/` folder into your Claude skills directory (typically `~/.claude/skills/` or the path configured in your Claude settings), then restart Claude.

## Author

Built using the [Claude Skill framework](https://github.com/anthropics/skills).
