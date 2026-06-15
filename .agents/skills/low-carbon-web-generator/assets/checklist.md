# Low-Carbon Website Verification Checklist

Run through this checklist after generating or refactoring a site.
All items should pass before marking the task complete.

---

## Byte Budgets

- [ ] Total page weight < 500 KB (ideal: < 200 KB)
- [ ] Client-side JS ≤ 50 KB gzipped per page
- [ ] CSS ≤ 20 KB gzipped per page
- [ ] No single image > 200 KB (after optimisation)

## Images

- [ ] All photos in AVIF or WebP format (JPEG fallback only)
- [ ] Responsive `srcset` with appropriate `sizes` attribute
- [ ] `loading="lazy"` on all below-fold images
- [ ] `fetchpriority="high"` on LCP image only
- [ ] Explicit `width` and `height` on all `<img>` elements
- [ ] Meaningful `alt` text (or `alt=""` for decorative)

## Fonts

- [ ] System fonts used where possible
- [ ] Web fonts self-hosted (not loaded from third-party CDN) or justified
- [ ] **Variable font** preferred over multiple static weights when available
- [ ] ≤ 3 font families and ≤ 2 weights per family
- [ ] **Total font files imported ≤ 6** (audit `@fontsource` imports)
- [ ] Every imported weight matches an actual `font-weight` use in CSS
- [ ] `font-display: swap` on all custom fonts
- [ ] Non-critical fonts (icons) deferred via `media="print"` trick
- [ ] Icon font replaced by inline SVG when ≤ 15 icons used per page
- [ ] FCP not increased by > 100ms from font loading

## JavaScript

- [ ] Zero JS on pages that don't need interactivity
- [ ] Progressive enhancement: core content works without JS
- [ ] Each `<script>` has documented justification and size estimate
- [ ] No `client:load` or `client:only` without explicit justification
- [ ] `<noscript>` fallback for critical JS-enhanced features
- [ ] Background timers/animations pause on `document.visibilityState === 'hidden'`
- [ ] No animation of `box-shadow` / `filter` / `backdrop-filter` on always-visible UI

## Third-Party Weight

- [ ] **One** analytics provider only (not GA + PostHog + Hotjar)
- [ ] Analytics deferred until `load` event or first user interaction
- [ ] No map iframes loaded eagerly — replaced with static image or click-to-load
- [ ] Video / Twitter / Instagram embeds use facade pattern (`lite-youtube` etc.)
- [ ] No live-fetched "carbon score" badges (use static screenshot)
- [ ] No chat widgets unless click-to-load
- [ ] No third-party CDN for assets that change rarely (self-host instead)

## CSS

- [ ] `prefers-reduced-motion` respected in global styles
- [ ] `prefers-contrast` considered for borders/outlines
- [ ] Design tokens centralised (not hardcoded colours/fonts)
- [ ] Scoped styles per component (no massive global stylesheet)
- [ ] No unused CSS rules shipped to production

## HTML & Accessibility

- [ ] Semantic elements (`<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>`)
- [ ] Single `<h1>` per page with proper heading hierarchy
- [ ] Skip link present: `<a href="#main-content">Skip to content</a>`
- [ ] All interactive elements keyboard-navigable
- [ ] Visible focus indicators (high-contrast focus rings)
- [ ] Colour contrast ≥ 4.5:1 (WCAG AA)
- [ ] ARIA attributes only when native semantics insufficient

## Caching & Deployment

- [ ] Hashed assets have `Cache-Control: immutable` headers
- [ ] HTML pages have `Cache-Control: must-revalidate` headers
- [ ] Security headers present (X-Frame-Options, X-Content-Type-Options, etc.)
- [ ] Static output pre-rendered (SSG, not SSR)
- [ ] Assets served from CDN edge (not origin)

## SEO + AEO

> Run the dedicated **`aeo-seo-optimizer`** skill checklist for the full set.
> Headline items also relevant to carbon (less wasted traffic = lower energy):

- [ ] Descriptive `<title>` and `<meta name="description">` per page
- [ ] `<link rel="canonical">` per page
- [ ] `sitemap.xml` generated at build time
- [ ] `robots.txt` declares an explicit AI bot policy
- [ ] JSON-LD structured data inline (no external request)
- [ ] `llms.txt` present (reduces wasted re-crawls by AI engines)

## Sustainability Transparency

- [ ] `carbon.txt` present at `/carbon.txt`
- [ ] `llms.txt` present at `/llms.txt` (recommended)
- [ ] Upstream services declared in `carbon.txt`
- [ ] Sustainability commitment documented (optional but recommended)

---

## Rating

After completing the checklist, rate the site:

| Score           | Rating        | Description                   |
| --------------- | ------------- | ----------------------------- |
| 100% items pass | 🏆 Exemplary  | Best-in-class low-carbon site |
| ≥ 90% pass      | 🟢 Excellent  | Minor gaps only               |
| ≥ 75% pass      | 🟡 Good       | Some optimisation needed      |
| ≥ 50% pass      | 🟠 Fair       | Significant opportunities     |
| < 50% pass      | 🔴 Needs work | Major refactoring required    |
