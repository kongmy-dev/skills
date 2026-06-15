# Universal Low-Carbon Web Techniques

Framework-agnostic practices for reducing the carbon footprint of any website.

---

## 1. Image Optimisation

Images are typically the largest contributor to page weight.

### Format priority

```
AVIF → WebP → JPEG (fallback)
Never use PNG for photos. Use SVG for icons/logos.
```

### Responsive delivery

```html
<picture>
  <source
    srcset="/hero-400.avif 400w, /hero-800.avif 800w, /hero-1200.avif 1200w"
    sizes="(max-width: 768px) 100vw, 800px"
    type="image/avif"
  />
  <source
    srcset="/hero-400.webp 400w, /hero-800.webp 800w, /hero-1200.webp 1200w"
    sizes="(max-width: 768px) 100vw, 800px"
    type="image/webp"
  />
  <img
    src="/hero-800.jpg"
    alt="Descriptive alt text"
    loading="lazy"
    decoding="async"
    width="800"
    height="450"
  />
</picture>
```

### Key rules

| Rule                                                    | Why                                               |
| ------------------------------------------------------- | ------------------------------------------------- |
| Always set `width` and `height` attributes              | Prevents CLS (Cumulative Layout Shift)            |
| Use `loading="lazy"` for below-fold images              | Avoids downloading bytes the user may never see   |
| Use `fetchpriority="high"` on LCP image only            | Speeds up the largest contentful paint            |
| Provide explicit `alt` text (or `alt=""` if decorative) | Accessibility + reduces need for fallback content |

---

## 2. Font Strategy

Web fonts add HTTP requests and bytes. Minimise their impact.

### Preference order

1. **System font stack** — zero transfer cost
2. **Variable web font, self-hosted** — one file covers all weights
3. **Self-hosted static subsets** — only the weights you actually use
4. **Google Fonts with `display=swap`** — last resort; third-party dependency

### System font stack

```css
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
```

### Self-hosted fonts

- Prefer `@fontsource-variable/<name>` over multiple `@fontsource/<name>/{weight}.css` imports
- Import only the weights you actually use — audit with:
  ```bash
  rg -o 'font-(thin|extralight|light|normal|medium|semibold|bold|extrabold|black)' src
  ```
- Set `font-display: swap` to avoid invisible text

### Defer non-critical font CSS

```html
<!-- Defer icon fonts: load as print, swap to all on load -->
<link rel="stylesheet" href="/icon-font.css" media="print" onload="this.media='all'" />
<noscript><link rel="stylesheet" href="/icon-font.css" /></noscript>
```

### Hard budgets

| Limit                                | Reason                                   |
| ------------------------------------ | ---------------------------------------- |
| ≤ 3 font families                    | Each adds discovery + handshake cost     |
| ≤ 2 weights per family               | Beyond that, switch to a variable font   |
| ≤ 6 total font files imported        | Above this, FCP slips past 100ms penalty |
| Subset to Latin (or required script) | Cuts 60–80% of file size                 |

### Anti-pattern

Importing every weight "just in case":

```ts
// ❌ 9 imports — ~270 KB
import '@fontsource/source-sans-3/300.css';
import '@fontsource/source-sans-3/400.css';
import '@fontsource/source-sans-3/500.css';
import '@fontsource/source-sans-3/600.css';
// + same pattern for serif + mono…

// ✅ 1 import — ~50 KB, covers full weight range
import '@fontsource-variable/source-sans-3';
```

If a project still uses static weights, drop everything that isn't matched
by an actual `font-weight` declaration. Most sites need 3: regular, semibold,
italic.

---

## 3. CSS Discipline

### Budget: ≤ 20 KB per page (gzipped)

### Rules

- **CSS over JS for visual effects**: transitions, animations, hover states, scroll-snap — all achievable in CSS
- **Scoped styles**: each component owns minimal CSS; no global framework bloat
- **Design tokens**: centralise colours, spacing, typography in CSS custom properties
- **Remove unused CSS**: tree-shake or audit regularly
- **Respect user preferences**:

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

@media (prefers-contrast: high) {
  :root {
    --border-width: 2px;
  }
}
```

---

## 4. JavaScript Minimisation

### Budget: ≤ 50 KB per page (gzipped)

### Decision tree

```
Does it need interactivity?
├── No  → Static HTML/CSS ✓  (0 KB)
└── Yes → Can CSS handle it?
    ├── Yes → CSS-only ✓  (0 KB)
    └── No  → Progressive enhancement needed
        ├── Small (<1 KB)  → Vanilla inline <script>
        ├── Reusable       → Web Component
        └── Complex state  → Lazy-hydrated framework component
```

### Progressive enhancement

- Core content works **without JavaScript**
- JS **enhances** — doesn't enable — the experience
- Always provide `<noscript>` fallbacks for critical functionality

### Size categories

| Category              | Size Limit | Example                      |
| --------------------- | ---------- | ---------------------------- |
| Critical (above fold) | 0 KB       | Static hero, nav links       |
| Enhancement           | < 5 KB     | Menu toggle, form validation |
| Feature               | 5–20 KB    | Carousel, accordion          |
| Full page app         | < 50 KB    | Complex interactive tool     |

---

## 5. HTML Best Practices

### Semantic structure

```html
<header>
  <!-- Site header -->
  <nav>
    <!-- Navigation -->
    <main>
      <!-- Primary content -->
      <article>
        <!-- Self-contained content -->
        <section>
          <!-- Thematic grouping -->
          <aside>
            <!-- Tangential content -->
            <footer><!-- Site footer --></footer>
          </aside>
        </section>
      </article>
    </main>
  </nav>
</header>
```

### Minimise DOM depth

- Avoid wrapper `<div>` elements when a semantic element exists
- Target ≤ 1,500 DOM elements per page
- Use `<Fragment>` (in frameworks) to avoid extra wrappers

### Skip links

```html
<a href="#main-content" class="skip-link">Skip to content</a>
```

---

## 6. Caching Strategy

Effective caching eliminates repeat transfers entirely.

### Headers

```
# Hashed build assets — cache forever
/_astro/*
  Cache-Control: public, max-age=31536000, immutable

# Static brand assets
/favicon.ico
  Cache-Control: public, max-age=31536000, immutable

# HTML pages — always revalidate
/*.html
  Cache-Control: public, max-age=0, must-revalidate
```

### Principles

- **Immutable for hashed files**: build tools add content hashes to filenames — cache for 1 year
- **Revalidate for HTML**: ensures content updates propagate immediately
- **CDN edge serving**: serve static assets from the nearest edge node to the user

---

## 7. Security Headers (Zero Runtime Cost)

Security headers don't add bytes to the response body and cost nothing at runtime.

```
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

---

## 8. Sustainability Transparency

### carbon.txt

Publish a machine-readable sustainability disclosure at `/carbon.txt`:

```toml
version = "0.4"
last_updated = "2026-01-01"

[org]
disclosures = [
  { doc_type = "web-page", url = "https://example.com/sustainability", title = "Our Sustainability Commitment" }
]

[upstream]
services = [
  { domain = "cloudflare.com", service_type = ["cdn", "shared-hosting"] }
]
```

### llms.txt

For AI discoverability, provide a structured summary at `/llms.txt` that reduces unnecessary crawling and re-rendering.

---

## 9. Third-Party Weight

Third-party scripts and embeds are usually the biggest opportunity on an
otherwise-lean site. See [third-party-weight.md](third-party-weight.md) for
patterns. Highlights:

- Replace map iframes (~700 KB) with static map images.
- Pick **one** analytics provider, not two.
- Use `lite-youtube`-style facades for video / social embeds.
- Inline SVGs when you need ≤ 15 icons; drop the icon font.
- Self-host (or screenshot) any "carbon score" badge instead of loading it live.

## 10. Runtime Energy

Bytes transferred isn't the only carbon dimension; CPU and battery on the
client also count. Cheap wins:

- **Pause work on hidden tabs**:
  ```ts
  let intervalId: number | undefined;
  const start = () => {
    intervalId ??= setInterval(tick, 3000);
  };
  const stop = () => {
    if (intervalId) {
      clearInterval(intervalId);
      intervalId = undefined;
    }
  };
  document.addEventListener('visibilitychange', () => {
    document.visibilityState === 'visible' ? start() : stop();
  });
  start();
  ```
- **Avoid `backdrop-filter` on always-visible elements** (sticky headers,
  modals). Each painted frame triggers a compositor pass. Replace with a
  solid background or a subtle border on always-visible UI.
- **Avoid `setInterval` chains in idle UI.** Use `IntersectionObserver` /
  user-event handlers instead.
- **Don't animate CPU-expensive properties** (`box-shadow`, `filter`).
  Animate `transform` and `opacity` only.

## 11. SEO

For SEO and AEO patterns (canonical, structured data, llms.txt, AI bot
policy), see the **`aeo-seo-optimizer`** skill — that scope is intentionally
out of this skill to keep responsibilities clean.

---

## 12. Measuring Carbon Impact

### Qualitative assessment

| Transfer size | Rating       | Description           |
| ------------- | ------------ | --------------------- |
| < 200 KB      | 🟢 Excellent | Minimal footprint     |
| 200–500 KB    | 🟡 Good      | Room for improvement  |
| 500 KB–1 MB   | 🟠 Fair      | Consider optimisation |
| > 1 MB        | 🔴 Poor      | Urgent action needed  |

### Per-asset breakdown

After each build, audit:

```bash
# List all output files sorted by size
find dist -type f | xargs ls -lhS
```

Track: HTML, CSS, JS, images, fonts as separate categories.
