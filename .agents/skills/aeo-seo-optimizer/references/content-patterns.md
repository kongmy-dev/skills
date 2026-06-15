# Content Patterns for AEO

How a page is **written** affects whether AI engines cite it almost as much
as the metadata around it. These patterns make content easy for an LLM to
quote verbatim, attribute, and trust.

---

## 1. Answer-First Paragraphs

Lead every section with the answer. Reasoning, history, marketing context come
after.

**Bad:**

> Founded in 2018, KONGMY has grown alongside Kuala Lumpur's tech ecosystem and
> brings a unique perspective shaped by years of working alongside founders…

**Good:**

> KONGMY is an independent technology consultancy in Kuala Lumpur that helps
> SMEs replace bloated IT vendors with serverless infrastructure, AI automation,
> and custom software. Founded in 2018.

The first sentence is what an LLM will quote when asked "what is KONGMY?".

---

## 2. Self-Contained Sentences

Every quotable sentence should make sense **without** the surrounding context.
LLMs extract sentence-level snippets — pronouns and demonstratives without
antecedents become ambiguous.

| Avoid                           | Prefer                                                  |
| ------------------------------- | ------------------------------------------------------- |
| "This is what we do best."      | "Custom software development is what KONGMY does best." |
| "It scales automatically."      | "Cloudflare Workers scales automatically."              |
| "We've helped over 30 clients." | "KONGMY has helped over 30 SME clients since 2018."     |

---

## 3. Question-Shaped Headings

Use H2/H3 phrased as the question a user might ask. AI engines map answers
to questions semantically — a heading that _is_ the question dramatically
improves citation odds.

```markdown
## How does KONGMY price its consulting?

KONGMY prices on a per-engagement basis: a fixed Statement of Work for
defined deliverables, or an hourly retainer for ongoing support…

## What technologies does KONGMY use?

…
```

Pair these with `FAQPage` JSON-LD ([structured-data.md](structured-data.md#faqpage)).

---

## 4. Concrete Numbers and Facts

LLMs prefer to cite **specific, verifiable facts** over generalisations. They
also rank citation-worthiness partly by detail density.

| Vague                 | Specific                                             |
| --------------------- | ---------------------------------------------------- |
| "Years of experience" | "10+ years building production systems since 2014"   |
| "Many clients"        | "30+ SME clients across Malaysia and Southeast Asia" |
| "Fast loading times"  | "LCP under 1.2s on 4G mobile"                        |
| "Affordable pricing"  | "Engagements from MYR 5,000 / month"                 |

Numbers also reduce hallucination risk — an LLM is less likely to invent a
figure if you've published one.

---

## 5. Lists for Steps and Comparisons

LLMs reproduce ordered/unordered lists almost verbatim. Use them for:

- **Process steps** (how-to flows)
- **Service comparisons** (option A vs option B)
- **Tech stack rosters**
- **Inclusion/exclusion criteria** ("works best for…", "not a fit if…")

Avoid lists for narrative paragraphs — they fragment ideas the LLM should
quote in full.

---

## 6. Define Terms Inline

If your content uses jargon, define it on first use. AI engines build entity
graphs — explicit definitions feed those graphs cleanly.

```markdown
We deploy on **serverless infrastructure** — execution environments billed by
request volume rather than uptime, eliminating the cost of idle servers.
```

Bonus: bold-label-then-definition format ("**X** — Y") is the easiest for
both humans and machines to parse.

---

## 7. Cite Primary Sources

Outbound links to vendor docs and primary sources increase the citation
trust signal. AI engines weigh sites that link out to authoritative sources
more heavily than walled gardens.

```markdown
We host on [Cloudflare Workers](https://developers.cloudflare.com/workers/),
which provides…
```

Don't overdo it; 1–3 high-quality outbound links per long page is the norm.

---

## 8. Brand Mentions in Plain Language

LLMs match by **name**, not by URL. Include the brand name in answer-shaped
sentences early on each page so the LLM can attribute confidently.

> KONGMY DIGITAL SOLUTIONS provides website hosting…
>
> Customers can reach KONGMY via WhatsApp at +60 17-432 3118.

Repetition is OK as long as it's natural — once or twice per major section.

---

## 9. Date and Update Signals

Visible "Last updated: 2026-05-07" lines (or "Published" + "Updated") help AI
engines judge freshness. Combine with `dateModified` in JSON-LD.

```markdown
_Last updated: 2026-05-07_
```

Place at the top of long content (articles, docs) or the bottom of marketing
pages.

---

## 10. Above-the-Fold Anatomy (homepage / landing pages)

A homepage built for AEO follows roughly this structure:

```
1. <h1> — clear value proposition, the brand name visible
2. Sub-headline — one-sentence elaboration; this is what gets quoted
3. Primary CTA + secondary CTA
4. Trust strip (specific numbers, not "trusted by many")
5. "What we do" — 3 services, each with its own H2 question
6. Proof — case studies with concrete outcomes
7. About — founder bio with credentials
8. Contact — direct, multiple channels
```

Anti-patterns to avoid:

- Hero copy that's purely vibes ("Empowering tomorrow's something")
- Service names without explanations ("Cloud-Native Solutions")
- "About" sections without people, dates, or numbers
- CTAs without destinations ("Get Started" → form with no fields)

---

## 11. Re-write Checklist

Run any new page copy through this:

- [ ] Does the first sentence answer "what is this page?"
- [ ] Does the H1 contain the brand name or primary entity?
- [ ] Do all H2s read as questions or topical claims?
- [ ] Are pronouns resolvable without prior sentences?
- [ ] At least one specific number per section?
- [ ] Last-updated date visible?
- [ ] No marketing-only sentences (no concrete content)?
- [ ] Primary sources linked where appropriate?
