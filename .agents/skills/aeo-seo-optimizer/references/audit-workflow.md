# Audit Workflow

Step-by-step process for auditing a site's SEO + AEO posture and producing a
prioritised findings list. The output is a punch-list, not auto-applied fixes
— most SEO/AEO changes are content decisions the user must make.

---

## Phase 1: Crawl Surface

Verify what crawlers can reach.

| Check                                                    | How                                                               |
| -------------------------------------------------------- | ----------------------------------------------------------------- |
| `/robots.txt` returns 200, `Content-Type: text/plain`    | `curl -I https://example.com/robots.txt`                          |
| Sitemap referenced and reachable                         | Open `Sitemap:` URL listed in robots.txt                          |
| Sitemap lists every canonical URL, no 404s, no redirects | `curl https://example.com/sitemap.xml \| grep -o 'https://[^<]*'` |
| AI bot policy is explicit (not implicit)                 | Search robots.txt for `GPTBot`, `ClaudeBot`, `PerplexityBot`      |
| `/llms.txt` and `/llms-full.txt` exist                   | `curl -I https://example.com/llms.txt`                            |

---

## Phase 2: On-Page Meta

For each unique page (homepage, every service, every legal page):

| Check                                                 | Pass criteria                                                |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| `<title>` unique and ≤ 60 chars                       | View source                                                  |
| `<meta name="description">` 120–160 chars             | View source                                                  |
| `<link rel="canonical">` present and self-referencing | Matches the requested URL (modulo trailing slash policy)     |
| `<html lang>` set                                     | Should be `en`, `en-US`, etc.                                |
| `<meta name="robots">` correct for the page           | `noindex` only on staging / thank-you / login pages          |
| Open Graph block complete                             | type, url, title, description, image (with width/height/alt) |
| Twitter card block complete                           | card type + image + alt                                      |
| Favicon set served                                    | favicon.ico, 16/32, apple-touch, manifest                    |
| `<link rel="alternate" hreflang>` for i18n            | If multilingual                                              |

Common findings:

- Multiple pages sharing a description ("default" template)
- Canonical pointing at home from internal pages
- OG image as WebP (LinkedIn/Slack drop these)
- Missing `og:image:width`/`height`
- Deprecated `<meta name="keywords">` still present

---

## Phase 3: Structured Data

Validate every JSON-LD block:

```bash
# Manual: paste page source into https://validator.schema.org/
# Manual: also test on https://search.google.com/test/rich-results
```

| Check                                                    | Pass criteria                                 |
| -------------------------------------------------------- | --------------------------------------------- |
| Homepage has `Organization` (or `LocalBusiness` subtype) | Validates, includes name + url + logo         |
| Homepage has `WebSite` block                             | Includes `potentialAction` if site has search |
| Each Article/BlogPost has `Article` schema               | datePublished + dateModified + author         |
| Service pages have `Service` schema                      | Linked to provider Organization by `@id`      |
| FAQ sections have `FAQPage` schema                       | Answer text matches visible page              |
| Deep pages have `BreadcrumbList`                         | Position numbers consecutive, items reachable |
| All `@id` values stable URLs (not random)                | Consistent across pages                       |
| No errors in validator                                   | Warnings OK on optional fields                |

Common findings:

- `Organization` repeated on every page (should be once + `@id` references)
- `dateModified` missing or stale
- `Article` author is a string, not a `Person` object
- `Service` blocks without `provider` linkage
- `LocalBusiness` without `address`/`geo`/`openingHours`

---

## Phase 4: AEO Signals

| Check                                            | Pass criteria                                |
| ------------------------------------------------ | -------------------------------------------- |
| `llms.txt` exists, follows spec                  | H1 + blockquote + sectioned link list        |
| `llms-full.txt` exists, mirrors site copy        | Clean prose, no nav/footer noise             |
| `<link rel="llms" href="/llms.txt">` in `<head>` | Discovery hint                               |
| `FAQPage` schema present where Q&A exists        | Verbatim answers in HTML                     |
| `Speakable` markers for short factual passages   | CSS selectors point to real elements         |
| Hero copy: answer-first sentence                 | Brand name + value proposition in first line |
| Headings phrased as questions where possible     | Especially for product/service explanations  |
| `dateModified` and visible "last updated"        | On homepage if marketing-led                 |
| AI bot allow-list explicit in robots.txt         | At minimum: GPTBot, ClaudeBot, PerplexityBot |

Common findings:

- llms.txt is just a contact card, no service descriptions
- llms.txt and llms-full.txt don't reference each other
- FAQPage missing despite an obvious Q&A section
- Hero h1 is decorative ("Build the future") with no entity binding
- robots.txt gives no AI policy — implicit allow

---

## Phase 5: Performance (CWV)

Performance is a ranking signal in classic SEO and a quality signal in AEO.

| Metric                          | Threshold (good) | How to measure                                              |
| ------------------------------- | ---------------- | ----------------------------------------------------------- |
| LCP (Largest Contentful Paint)  | < 2.5s           | Lighthouse, PageSpeed Insights, Chrome DevTools Performance |
| INP (Interaction to Next Paint) | < 200ms          | Same                                                        |
| CLS (Cumulative Layout Shift)   | < 0.1            | Same                                                        |
| TTFB (Time to First Byte)       | < 800ms          | `curl -w '%{time_starttransfer}'`                           |
| Total page weight               | < 500 KB         | DevTools → Network → "Disable cache" + reload               |

Common offenders:

- Unoptimised images (PNG/JPG instead of AVIF/WebP)
- Render-blocking third-party scripts (analytics, chat widgets)
- Web fonts blocking text paint
- `<iframe>` embeds (Google Maps, YouTube, Twitter) loaded eagerly
- Multiple analytics (e.g. GA + PostHog + Hotjar)

---

## Phase 6: Findings Output

Produce a table organised by priority. **Estimate effort** so the user can
sequence work.

| #   | Priority | Area        | Finding                                        | Suggested fix                            | Effort |
| --- | -------- | ----------- | ---------------------------------------------- | ---------------------------------------- | ------ |
| 1   | P0       | AEO         | No `FAQPage` schema; site has visible Q&A      | Add JSON-LD `FAQPage` to home + services | 30 min |
| 2   | P0       | SEO         | `og:image` is WebP — LinkedIn/Slack drop it    | Generate 1200×630 PNG for OG             | 30 min |
| 3   | P1       | Performance | Google Maps iframe loads ~500 KB on every page | Replace with click-to-load static map    | 1–2 hr |
| 4   | P1       | AEO         | robots.txt has no explicit AI bot policy       | Add allow-list for major AI crawlers     | 5 min  |
| …   | …        | …           | …                                              | …                                        | …      |

### Priority guide

- **P0 — Blocking**: visible bug, missing critical schema, deprecated/incorrect tags.
- **P1 — High value, low risk**: AEO additions, performance wins ≥ 100 KB.
- **P2 — Nice to have**: micro-optimisations, optional schema types, refinements.

### Output template

```markdown
## SEO + AEO Audit — {Site Name} ({date})

### Summary

{1–2 sentence assessment of overall posture}

### Strengths

- {What's already correct}

### Findings (prioritised)

| #   | Priority | Area | Finding | Fix | Effort |
| --- | -------- | ---- | ------- | --- | ------ |
| 1   | P0       | …    | …       | …   | …      |

### Recommended next steps

1. {Top 3 in execution order}
```

---

## Phase 7: Hand-Off

Do **not** auto-apply most fixes. Reasons:

- Description / title rewrites are brand decisions.
- Bot policy choices have legal/strategic implications.
- FAQ content must be authentic, not invented.

**Auto-applicable safely:** structured data scaffolding, missing canonical
links, missing OG dimensions, sitemap generation. Always confirm before
shipping.
