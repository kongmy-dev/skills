# llms.txt and llms-full.txt

`llms.txt` is a proposed convention (Jeremy Howard, late 2024) for giving
LLMs a hand-curated, low-noise summary of a site. Adoption is voluntary but
growing: Anthropic, Cloudflare, Vercel, Stripe, and many docs sites publish
one. Authoring it is a high-leverage AEO move — small file, large impact on
how AI engines describe and cite your site.

Spec: <https://llmstxt.org>

---

## 1. The Two Files

| File             | Purpose                                                                        | Size guide |
| ---------------- | ------------------------------------------------------------------------------ | ---------- |
| `/llms.txt`      | **Index** — short summary + curated list of canonical URLs an LLM should read. | ≤ 2 KB     |
| `/llms-full.txt` | **Full content** — concatenated long-form text of the canonical pages.         | ≤ 100 KB   |

Both go in `public/` and are served as `text/plain` (or `text/markdown`).
Add `Cache-Control: public, max-age=3600` — content changes are infrequent
but benefit from a quick TTL when they do.

---

## 2. llms.txt Format

The spec is **markdown with strict structural rules**:

```markdown
# Site or Product Name

> One-paragraph blockquote summary. Aim for 1–3 sentences. This is what an LLM
> will quote when asked "what is X?". Be factual, brand-aware, and concise.

Optional supplementary paragraphs of context. Avoid marketing fluff — describe
what the site is and what a user can find here.

## Section heading (e.g. Services, Docs, API)

- [Page Title](https://example.com/page): One-line description of what's here.
- [Another Page](https://example.com/other): One-line description.

## Optional

- [Less critical link](https://example.com/changelog): Things that aren't core.
```

### Rules

- **First line: `# Site Name` H1.** Required.
- **Second block: a blockquote `> …`** with the summary. Required.
- **Section headings: `##` H2.** No deeper nesting at the top level.
- **Each entry: a markdown link followed by `: description`.** No nested lists.
- **An "Optional" section** at the end signals less-critical links.
- All URLs absolute.

### Example (this project)

```markdown
# KONGMY Digital Solutions

> KONGMY DIGITAL SOLUTIONS is an independent, founder-led technology consultancy
> serving SMEs in Kuala Lumpur, Malaysia. We build bespoke internal tools, AI &
> WhatsApp automation, and lean serverless infrastructure without typical agency
> overhead.

## Services

- [Website & App Hosting](https://kongmy.dev/#services): Reliable hosting that doesn't require a dedicated IT person.
- [AI & WhatsApp Automation](https://kongmy.dev/#services): Conversational interfaces wired into business systems.
- [Custom Software & Integrations](https://kongmy.dev/#services): Internal tools, customer portals, system glue.

## About

- [About the Founder](https://kongmy.dev/#about): Background, credentials, technologies.
- [Track Record](https://kongmy.dev/#track-record): Recent client outcomes.

## Reference

- [Full Site Content](https://kongmy.dev/llms-full.txt): Long-form copy of every page.
- [Sustainability Disclosure](https://kongmy.dev/carbon.txt): Hosting and energy footprint.

## Contact

- [WhatsApp](https://wa.me/60174323118): +60 17-432 3118
- [Email](mailto:hello@kongmy.dev): hello@kongmy.dev

## Optional

- [Privacy Policy](https://kongmy.dev/privacy)
- [Terms of Engagement](https://kongmy.dev/terms)
```

---

## 3. llms-full.txt Format

No strict spec. Treat it as a **single concatenated markdown document**:

```markdown
# Site Name

> Same blockquote summary as llms.txt.

## Page Title 1

Full text of page 1, kept readable. Strip nav/footer/cookie boilerplate.

## Page Title 2

Full text of page 2.
```

### Rules

- **Scrub navigation, footer, cookie banners, analytics text.**
- **Preserve internal headings** (`##`, `###`) so an LLM can cite sections.
- **Use absolute URLs** for any references.
- **Cap at ~100 KB.** Anything bigger and LLMs may truncate or skip it.

For a marketing site: 1–10 pages of clean copy. For docs: top-level concepts
plus the most-cited reference pages.

---

## 4. Discovery

Add a `<link>` hint in every page's `<head>`:

```html
<link rel="llms" href="/llms.txt" />
```

This is **not** a registered link relation yet (so validators may complain),
but Anthropic's own crawler and several others honour it. Cost: 1 line.

You can also reference llms.txt from `robots.txt`:

```
# AI discovery
LLMs: https://example.com/llms.txt
```

This is a custom directive — bots that don't understand it ignore it; bots
that do (Anthropic, OpenAI experimental) follow it.

---

## 5. When NOT to Publish

- Sites under active stealth or NDA — the file will be crawled and quoted.
- Sites that monetise content paywalls — `llms-full.txt` would leak it.
- Sites that frequently change copy — keeping the file in sync is overhead.

In all other cases, the marginal upside (citations, accurate brand summaries)
outweighs the cost.

---

## 6. Authoring Tips

- Write the blockquote summary as if **answering "what is this site?" in 30
  seconds** to a stranger.
- Lead with **what the user gets**, not the company history.
- Keep link descriptions to **one sentence each**.
- Refresh quarterly or whenever services, pricing, or location change.
- If you have a blog/docs site, list **only canonical / evergreen URLs** —
  not every post.
