# SEO + AEO Verification Checklist

Run after scaffolding or refactoring a site for search/answer-engine
visibility. Sections map to the references in the parent skill.

---

## Crawl Surface

- [ ] `/robots.txt` returns 200 with `Content-Type: text/plain`
- [ ] `Sitemap:` directive in `robots.txt` points at a reachable XML index
- [ ] All sitemap entries return 200 (no redirects, no 404s)
- [ ] AI bot policy is explicit (allow-list or block-list, not implicit)
- [ ] `/llms.txt` exists and follows spec
- [ ] `/llms-full.txt` exists and mirrors site copy
- [ ] `<link rel="llms" href="/llms.txt">` in `<head>`

## Per-Page Meta

- [ ] Unique `<title>` ≤ 60 chars
- [ ] Unique `<meta name="description">` 120–160 chars
- [ ] `<link rel="canonical">` present and self-referential
- [ ] `<html lang>` set
- [ ] `<meta name="robots">` correct (no accidental `noindex` on prod pages)
- [ ] No deprecated tags (`keywords`, `author` meta, `geo.*`, `revisit-after`, `distribution`)

## Open Graph + Twitter

- [ ] `og:type`, `og:url`, `og:title`, `og:description` all present
- [ ] `og:image` is **PNG or JPEG**, 1200×630
- [ ] `og:image:width`, `og:image:height`, `og:image:alt` all present
- [ ] `twitter:card` set to `summary_large_image`
- [ ] `twitter:image:alt` present

## Structured Data

- [ ] Homepage carries `Organization` (or appropriate `LocalBusiness` subtype)
- [ ] Homepage carries `WebSite` (with `potentialAction` if site has search)
- [ ] Articles/blog posts carry `Article` with `datePublished` + `dateModified` + `author`
- [ ] Service pages carry `Service` (linked to provider via `@id`)
- [ ] Q&A sections carry `FAQPage` (answers verbatim in HTML)
- [ ] Deep pages carry `BreadcrumbList`
- [ ] All blocks validate at <https://validator.schema.org/>
- [ ] No errors in <https://search.google.com/test/rich-results>

## AEO Signals

- [ ] First sentence of each page answers "what is this page?"
- [ ] H1 contains the brand name or primary entity
- [ ] H2/H3 phrased as questions where applicable
- [ ] Specific numbers/facts (not "many", "fast", "affordable")
- [ ] `Speakable` selectors point to short factual passages
- [ ] Visible "last updated" date on long-form pages
- [ ] llms.txt and llms-full.txt cross-reference each other

## Performance (Ranking + Quality Signal)

- [ ] LCP < 2.5s on 4G mobile
- [ ] INP < 200ms
- [ ] CLS < 0.1
- [ ] Total page weight < 500 KB (ideal: < 200 KB)
- [ ] No render-blocking third-party scripts above the fold

## Validation

- [ ] Google Search Console: no coverage errors
- [ ] Bing Webmaster Tools: site verified and indexed
- [ ] All structured data validators pass
- [ ] Canonical chosen consistently across nav, sitemap, JSON-LD
