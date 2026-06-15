---
name: low-carbon-web-generator
description: >-
  Generate websites with minimal carbon footprint using static-first architecture,
  aggressive byte reduction, and edge-optimised deployment. Supports three modes:
  generate from scratch, generate from a design mockup, or audit and refactor an
  existing codebase. Use when the user asks to build a "green website",
  "low-carbon site", "sustainable web project", "eco-friendly page",
  "carbon-conscious website", or wants to minimise transfer size, reduce page
  weight, or optimise for sustainability.
license: MIT
compatibility: Designed for Claude Code and similar agentic coding environments. Requires Node.js/Bun for scaffolding.
metadata:
  author: kongmy
  version: '1.1'
---

# Low-Carbon Web Generator

Build websites that are fast, accessible, and kind to the planet by minimising
bytes transferred, server CPU usage, and client-side computation.

## Quick Start — Choose a Mode

### Mode 1: Generate from Scratch

1. Ask the user for the site's **purpose**, **pages**, and **brand tokens** (colours, fonts, logo).
2. Scaffold a new Astro project with SSG output and Cloudflare Pages adapter.
   See [astro-patterns.md](references/astro-patterns.md) for the full project template.
3. Apply every technique from [carbon-techniques.md](references/carbon-techniques.md).
4. Configure deployment per [cloudflare-optimization.md](references/cloudflare-optimization.md).
5. Add `public/carbon.txt` using the [template](assets/carbon.txt.template).
6. Run the [verification checklist](assets/checklist.md) before marking complete.

### Mode 2: Generate from Design

1. Receive a screenshot, mockup, or design description from the user.
2. Decompose the design into semantic HTML sections and reusable components.
3. Implement each component following the **zero-JS default** principle:
   - Prefer CSS for animations, hover states, and layout.
   - Use vanilla `<script>` only for genuine interactivity (menu toggles, form enhancement).
4. Optimise every image asset: convert to AVIF/WebP, generate responsive `srcset`, apply `loading="lazy"`.
5. Apply carbon techniques and verify with the checklist.

### Mode 3: Audit Existing Codebase

1. Read [codebase-analyzer.md](references/codebase-analyzer.md) for the full progressive-disclosure workflow.
2. **Inventory** — list all pages, components, scripts, stylesheets, images, and fonts.
3. **Measure** — calculate total transfer size per category.
4. **Identify hotspots** — flag unoptimised images, render-blocking resources, excessive JS, missing cache headers.
5. **Prioritise** — rank fixes by carbon impact (biggest byte savings first).
6. **Refactor** — apply techniques incrementally from [carbon-techniques.md](references/carbon-techniques.md).
7. Verify each change with the checklist.

## Core Principles

These are **non-negotiable** for every generated page:

| #   | Principle                                                                   | Budget       |
| --- | --------------------------------------------------------------------------- | ------------ |
| 1   | **Static first (SSG)** — pre-render everything; SSR only with justification | —            |
| 2   | **Zero JS by default** — no client-side JS unless interactivity requires it | ≤ 50 KB/page |
| 3   | **CSS over JS** — animations, transitions, and layout via CSS               | ≤ 20 KB/page |
| 4   | **Optimised images** — AVIF → WebP → JPEG, responsive srcset, lazy loading  | —            |
| 5   | **Font discipline** — system fonts first; ≤ 3 families, ≤ 2 weights each    | swap only    |
| 6   | **Aggressive caching** — immutable hashes for assets, revalidate for HTML   | —            |
| 7   | **Semantic HTML** — correct elements, minimal DOM depth, accessible         | WCAG AA      |
| 8   | **Transparency** — publish `carbon.txt`, document sustainability choices    | —            |

## Decision Tree — Adding Client-Side JavaScript

```
Does it need interactivity?
├── No  → Static HTML/CSS ✓
└── Yes → Can CSS handle it? (:hover, :target, animations)
    ├── Yes → CSS-only ✓
    └── No  → Progressive enhancement needed
        ├── Small (<1 KB) → Vanilla inline <script> ✓
        ├── Reusable       → Web Component ✓
        └── Complex state  → Framework component with client:idle
```

## Platform-Specific References

Load these **only when working with the corresponding platform**:

- **Astro**: [astro-patterns.md](references/astro-patterns.md) — SSG config, component patterns, image pipeline, font loading
- **Cloudflare**: [cloudflare-optimization.md](references/cloudflare-optimization.md) — wrangler config, edge caching, security flags
- **Any framework**: [carbon-techniques.md](references/carbon-techniques.md) — universal byte-reduction and sustainability techniques
- **Existing codebase**: [codebase-analyzer.md](references/codebase-analyzer.md) — progressive audit and refactoring workflow
- **Common heavy dependencies** (analytics, maps, social embeds, badges): [third-party-weight.md](references/third-party-weight.md) — patterns for cutting external bytes without losing the feature

## Companion Skill

For SEO and AEO concerns (canonical URLs, structured data, llms.txt, AI bot
policy, FAQ schema, OG image rules), use the **`aeo-seo-optimizer`** skill.
Don't reimplement those patterns here — this skill stays focused on byte
reduction and runtime energy.

## Verification

After generating or refactoring, run through the [verification checklist](assets/checklist.md).

Key checks:

- [ ] Total page weight under 500 KB (ideal: under 200 KB)
- [ ] Client JS ≤ 50 KB gzipped per page (incl. analytics)
- [ ] CSS ≤ 20 KB gzipped per page
- [ ] All images in AVIF or WebP with responsive srcset
- [ ] No more than **one** analytics provider on the page
- [ ] No third-party iframes loaded above the fold (Maps, YouTube, Tweets — use click-to-load)
- [ ] Total imported font files ≤ 6 (≤ 3 families × ≤ 2 weights, +italic only when used)
- [ ] `carbon.txt` present at `/carbon.txt`
- [ ] `prefers-reduced-motion` respected
- [ ] Background timers/animations pause on `document.visibilityState === 'hidden'`
- [ ] Lighthouse Performance score ≥ 90
