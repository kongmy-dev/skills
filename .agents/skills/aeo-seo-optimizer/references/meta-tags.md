# Meta Tags Reference

A canonical `<head>` block that satisfies both SEO and AEO crawlers without
shipping deprecated or redundant tags.

---

## 1. The Minimum Viable Block

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />

    <!-- Identity -->
    <title>{Page Title — ≤ 60 chars including brand}</title>
    <meta
      name="description"
      content="{Concise summary, 120–160 chars, answers 'what is this page'}"
    />

    <!-- Canonical & language -->
    <link rel="canonical" href="https://example.com/path" />
    <link rel="alternate" hreflang="en" href="https://example.com/path" />
    <link rel="alternate" hreflang="x-default" href="https://example.com/path" />

    <!-- Crawl directives -->
    <meta
      name="robots"
      content="index, follow, max-image-preview:large, max-snippet:-1, max-video-preview:-1"
    />

    <!-- Discovery -->
    <link rel="sitemap" type="application/xml" href="/sitemap-index.xml" />
    <link rel="alternate" type="application/rss+xml" href="/feed.xml" />
    <!-- if you have a feed -->
    <link rel="llms" href="/llms.txt" />
    <!-- AEO discovery hint -->
  </head>
  …
</html>
```

### Title rules

- **Each page must be unique.** Duplicate titles are the most common SEO bug.
- **≤ 60 chars (incl. spaces and brand).** Google truncates around 580px which
  is roughly 60 chars for typical fonts.
- **Pattern**: `Primary Phrase — Differentiator | Brand`. Put the keyword first.
- **Don't keyword-stuff.** Modern search ranks topical relevance; stuffing hurts.

### Description rules

- **120–160 chars** (mobile sometimes truncates around 120).
- **Answer the page's question** in one sentence; AI engines often quote it.
- Don't repeat the title verbatim.
- Avoid empty boilerplate ("Welcome to …").

---

## 2. Open Graph (Social + LinkedIn + Slack)

```html
<meta property="og:type" content="website" />
<!-- "article" for blog posts, "product" for e-commerce -->
<meta property="og:site_name" content="Brand Name" />
<meta property="og:locale" content="en_US" />
<meta property="og:url" content="https://example.com/path" />
<meta property="og:title" content="{Same as <title> or shorter}" />
<meta property="og:description" content="{Same as meta description}" />
<meta property="og:image" content="https://example.com/og-image.png" />
<meta property="og:image:type" content="image/png" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:image:alt" content="{Describe what's in the image}" />
```

### OG image rules

- **1200 × 630 px** is the canonical size — fits Twitter, LinkedIn, Facebook,
  Slack, Discord, iMessage previews.
- **PNG or JPEG, never WebP/AVIF.** Many platforms (LinkedIn, Slack, iMessage)
  silently drop modern formats from previews.
- Always include `og:image:width`/`height` — without them some platforms
  refuse to render the card.
- Always include `og:image:alt` for screen readers.
- Keep it ≤ 300 KB; Slack will refuse to fetch larger images.

---

## 3. Twitter / X Card

```html
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:url" content="https://example.com/path" />
<meta name="twitter:title" content="{Title}" />
<meta name="twitter:description" content="{Description}" />
<meta name="twitter:image" content="https://example.com/og-image.png" />
<meta name="twitter:image:alt" content="{Describe what's in the image}" />
<!-- Optional, only if you have an X account: -->
<meta name="twitter:site" content="@brand" />
<meta name="twitter:creator" content="@author" />
```

`summary_large_image` is the right card type for almost everyone. The
small-image `summary` card is harder to spot in feeds.

---

## 4. Robots Directives

```html
<meta
  name="robots"
  content="index, follow, max-image-preview:large, max-snippet:-1, max-video-preview:-1"
/>
```

| Directive                 | Effect                                                              |
| ------------------------- | ------------------------------------------------------------------- |
| `index, follow`           | Standard. Use `noindex, nofollow` for staging, drafts, login pages. |
| `max-image-preview:large` | Allows large image thumbnails in SERP — strongly recommended.       |
| `max-snippet:-1`          | No length limit on text snippets.                                   |
| `max-video-preview:-1`    | No length limit on video previews.                                  |
| `noarchive`               | Prevents cached copies. Rarely useful.                              |
| `nosnippet`               | Hides snippets entirely. Bad for CTR.                               |

For per-bot control, use `X-Robots-Tag` HTTP headers or specific user agents
(see [robots-and-bots.md](robots-and-bots.md)).

---

## 5. Favicon Set

```html
<link rel="icon" type="image/x-icon" href="/favicon.ico" />
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png" />
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png" />
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png" />
<link rel="manifest" href="/site.webmanifest" />
```

Generate with [realfavicongenerator.net](https://realfavicongenerator.net) — it
also produces the `.webmanifest`.

---

## 6. Performance Hints

```html
<!-- Preconnect to third-party origins used above the fold -->
<link rel="preconnect" href="https://fonts.googleapis.com" crossorigin />

<!-- Preload the LCP image so the browser fetches it before parsing CSS -->
<link rel="preload" as="image" href="/hero.avif" fetchpriority="high" />

<!-- Defer non-critical CSS (icon fonts, print styles) -->
<link rel="stylesheet" href="/icons.css" media="print" onload="this.media='all'" />
```

Don't `preconnect` more than ~3 origins; each costs a TLS handshake budget.

---

## 7. What to Skip

These are widely cargo-culted but **add nothing or actively hurt**:

| Tag                                          | Why skip                                                             |
| -------------------------------------------- | -------------------------------------------------------------------- |
| `<meta name="keywords">`                     | Google ignored since ~2009; Bing ignores; Yandex partially uses.     |
| `<meta name="author">`                       | Use `Person` JSON-LD on articles instead.                            |
| `<meta name="generator">`                    | Pure noise.                                                          |
| `<meta name="distribution">`                 | Obsolete since the 2000s.                                            |
| `<meta name="revisit-after">`                | Never honoured by any major crawler.                                 |
| `<meta name="rating">`                       | Use `contentRating` in JSON-LD instead.                              |
| `<meta name="geo.*">` / `<meta name="ICBM">` | Google ignores; use `LocalBusiness` JSON-LD with `address`/`geo`.    |
| `<meta name="copyright">`                    | Use the structured `copyrightHolder` field on schema or page footer. |
| `og:image:type` (without width/height)       | Unhelpful alone; only useful alongside dimensions.                   |
| Multiple `<meta name="description">` tags    | Some bots concatenate, others pick the last — always inconsistent.   |

---

## 8. Per-Framework Cheatsheet

### Astro

```astro
---
const { title, description, image, canonicalUrl } = Astro.props;
const canonical = canonicalUrl ?? new URL(Astro.url.pathname, Astro.site).href;
const ogImage = new URL(image ?? '/og-image.png', Astro.site).href;
---

<head>
  <title>{title}</title>
  <meta name="description" content={description} />
  <link rel="canonical" href={canonical} />
  <meta property="og:title" content={title} />
  …
</head>
```

### Next.js (App Router)

```ts
// app/page.tsx
export const metadata = {
  title: '…',
  description: '…',
  alternates: { canonical: 'https://example.com/' },
  openGraph: { images: [{ url: '/og.png', width: 1200, height: 630, alt: '…' }] },
  robots: { index: true, follow: true, 'max-image-preview': 'large' },
};
```

### Plain HTML

Just author the block from §1 directly. Use a build step or include partial
to avoid drift.
