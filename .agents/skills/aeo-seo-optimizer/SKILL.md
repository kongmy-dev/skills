---
name: aeo-seo-optimizer
description: >-
  Optimise a website for both classic search engines (SEO) and AI answer engines
  (AEO / GEO / LLM-SEO). Three modes: scaffold meta + structured data on a new
  site, audit an existing site for gaps, or extend a site with AEO-specific
  signals (llms.txt, FAQPage, Speakable, WebSite.SearchAction). Use when the
  user asks about "SEO", "AEO", "answer engine optimisation", "LLM SEO",
  "structured data", "schema.org", "rich results", "Google snippet",
  "AI search visibility", "ChatGPT search", "Perplexity citations",
  "llms.txt", "robots.txt for AI", "meta tags", or "sitemap".
license: MIT
compatibility: Framework-agnostic. Examples target Astro/Next.js/static HTML; principles apply to any stack.
metadata:
  author: kongmy
  version: '1.0'
---

# AEO + SEO Optimiser

A unified skill for **classic SEO** (Google/Bing crawl + rank) and **AEO**
(answer engines like ChatGPT Search, Perplexity, Google AI Overviews, Claude
with web access). Both share the same foundation; AEO layers on additional
machine-readable signals.

## Definitions

| Term        | Meaning                                                                       |
| ----------- | ----------------------------------------------------------------------------- |
| **SEO**     | Optimisation for traditional crawlers and ranked SERP results.                |
| **AEO**     | "Answer Engine Optimisation" — being **cited** in AI-generated answers.       |
| **GEO**     | "Generative Engine Optimisation" — same idea, sometimes used interchangeably. |
| **LLM SEO** | Casual umbrella term for AEO + content patterns LLMs prefer.                  |

## Quick Start — Choose a Mode

### Mode 1: Scaffold (new site)

1. Build the [base meta block](references/meta-tags.md) into the root layout.
2. Generate sitemap + robots.txt; add explicit AI bot policy ([robots-and-bots.md](references/robots-and-bots.md)).
3. Add a homepage `Organization` / `LocalBusiness` JSON-LD plus a `WebSite`
   block with `potentialAction` ([structured-data.md](references/structured-data.md)).
4. Add `llms.txt` and `llms-full.txt` ([llms-txt.md](references/llms-txt.md)).
5. Run the [verification checklist](assets/checklist.md).

### Mode 2: Audit (existing site)

1. Run the [audit workflow](references/audit-workflow.md). It walks crawl
   surface, on-page meta, structured data, AEO signals, and CWV in that order.
2. Output a prioritised findings list (P0/P1/P2). **Do not auto-fix** unless
   the user asks — most SEO/AEO changes are content decisions.

### Mode 3: AEO upgrade (existing SEO-ready site)

1. Add `llms.txt` + `llms-full.txt` (see [llms-txt.md](references/llms-txt.md)).
2. Inject `FAQPage` schema for any Q&A content ([structured-data.md](references/structured-data.md#faqpage)).
3. Add `Speakable` markers for short factual passages a voice assistant could read.
4. Re-write hero / above-fold copy in **answer-first** style (see [content-patterns.md](references/content-patterns.md)).
5. Confirm `robots.txt` policy for `GPTBot`, `ClaudeBot`, `PerplexityBot`,
   `Google-Extended`, `CCBot`, `Bytespider`, `Amazonbot`.

## Core Principles

These hold for both SEO and AEO:

| #   | Principle                                                                      |
| --- | ------------------------------------------------------------------------------ |
| 1   | **One canonical URL per resource** — no duplicates, no parameter drift.        |
| 2   | **Crawlable static HTML** — JS-rendered content is fragile for AEO crawlers.   |
| 3   | **Semantic, factual prose** — answer the question in the first 1–2 sentences.  |
| 4   | **Structured data on every key page** — JSON-LD, validated against schema.org. |
| 5   | **Explicit policy for AI bots** — opt in or out clearly in robots.txt.         |
| 6   | **Freshness signals** — `dateModified` in JSON-LD, regenerated sitemap.        |
| 7   | **Citation-friendly** — short paragraphs, lists, headings, primary sources.    |
| 8   | **Performance is a ranking signal** — CWV (LCP/INP/CLS) and TBT still matter.  |

## Decision Tree — Which Schema?

```
What is the page about?
├── Company/brand homepage      → Organization (+ LocalBusiness if physical)
├── Service description         → Service (+ Offer if priced)
├── Article / blog post         → Article / BlogPosting (+ author Person)
├── FAQ / Q&A section           → FAQPage (questions+accepted answers)
├── How-to / tutorial           → HowTo
├── Product                     → Product (+ AggregateRating, Offer)
├── Event                       → Event
├── Person profile              → Person
├── Job posting                 → JobPosting
├── Recipe                      → Recipe
├── Multiple breadcrumbs        → BreadcrumbList (always add on deep pages)
└── Site-level (root)           → WebSite + SearchAction (potentialAction)
```

A single page can carry multiple JSON-LD blocks. Prefer one `@graph` array
when objects reference each other.

## Decision Tree — AI Bot Policy

```
Do you want your content cited in AI answers?
├── Yes → allow GPTBot, ClaudeBot, PerplexityBot, Google-Extended in robots.txt
│        + publish llms.txt + llms-full.txt
│        + add FAQPage / Speakable schema
└── No  → disallow them; consider an Allow-List for audit/quoting bots only
```

The right answer for marketing/lead-gen sites is usually **yes**. AI engines
become a discovery surface; opting out trades visibility for content control.

## Files Reference

Load **only what's relevant** to keep context lean:

- [meta-tags.md](references/meta-tags.md) — canonical, OG, Twitter, language, robots, geo (and what's deprecated).
- [structured-data.md](references/structured-data.md) — JSON-LD recipes for the common schema types.
- [llms-txt.md](references/llms-txt.md) — llms.txt / llms-full.txt format and authoring rules.
- [robots-and-bots.md](references/robots-and-bots.md) — robots.txt patterns + AI crawler matrix.
- [content-patterns.md](references/content-patterns.md) — writing for AEO: answer-first, citable, scannable.
- [audit-workflow.md](references/audit-workflow.md) — step-by-step audit and findings template.
- [assets/checklist.md](assets/checklist.md) — final verification checklist.

## Verification

Before marking a site complete, all items in [checklist.md](assets/checklist.md)
should pass. Headline checks:

- [ ] Every page has unique `<title>` (≤ 60 chars), description (≤ 160 chars), canonical.
- [ ] Homepage carries `Organization` + `WebSite`; key sections carry contextual schema.
- [ ] `robots.txt` declares an explicit AI bot policy.
- [ ] `sitemap.xml` is generated at build time and listed in `robots.txt`.
- [ ] `llms.txt` and `llms-full.txt` exist and link to one another.
- [ ] All structured data validates in <https://validator.schema.org/>.
- [ ] Core Web Vitals: LCP < 2.5s, INP < 200ms, CLS < 0.1.
