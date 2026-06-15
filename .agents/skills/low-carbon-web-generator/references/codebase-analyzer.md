# Codebase Analyser — Progressive Audit Workflow

Step-by-step guide for auditing an existing website's carbon footprint and
refactoring it incrementally. Uses **progressive disclosure** to avoid
overwhelming changes.

---

## Phase 1: Inventory

List everything the site ships to the browser.

### Files to catalogue

```
Pages:        src/pages/**/*.{astro,html,jsx,tsx}
Components:   src/components/**/*
Layouts:      src/layouts/**/*
Stylesheets:  src/styles/**/* + inline <style> blocks
Scripts:      src/**/*.{js,ts} + inline <script> blocks
Images:       public/**/*.{png,jpg,jpeg,gif,svg,webp,avif}
              src/assets/**/*.{png,jpg,jpeg,gif,svg,webp,avif}
Fonts:        public/**/*.{woff,woff2,ttf,eot}
              + @fontsource imports
              + Google Fonts <link> tags
Data:         src/data/**/*
Config:       astro.config.*, wrangler.*, _headers, _redirects
```

### Output format

Create a table:

| Category | File                        | Size   | Notes               |
| -------- | --------------------------- | ------ | ------------------- |
| Image    | public/hero.png             | 2.1 MB | ❌ Not optimised    |
| Font     | public/fonts/Inter.woff2    | 45 KB  | ✅ Subsetted        |
| Script   | src/components/Carousel.tsx | 12 KB  | ⚠️ Uses client:load |
| …        | …                           | …      | …                   |

---

## Phase 2: Measure

### Per-page transfer budget

Build the site and measure output:

```bash
# Build and measure total output
bun run build
du -sh dist/

# List files by size (largest first)
find dist -type f -exec ls -lhS {} + | head -40

# Separate by type
echo "=== HTML ===" && find dist -name "*.html" -exec du -ch {} + | tail -1
echo "=== CSS ===" && find dist -name "*.css" -exec du -ch {} + | tail -1
echo "=== JS ===" && find dist -name "*.js" -exec du -ch {} + | tail -1
echo "=== Images ===" && find dist \( -name "*.png" -o -name "*.jpg" -o -name "*.webp" -o -name "*.avif" -o -name "*.svg" \) -exec du -ch {} + | tail -1
echo "=== Fonts ===" && find dist \( -name "*.woff" -o -name "*.woff2" \) -exec du -ch {} + | tail -1
```

### Record baseline

| Category  | Current Size | Budget       | Status |
| --------- | ------------ | ------------ | ------ |
| HTML      | ? KB         | —            |        |
| CSS       | ? KB         | ≤ 20 KB      |        |
| JS        | ? KB         | ≤ 50 KB      |        |
| Images    | ? KB         | Minimised    |        |
| Fonts     | ? KB         | Subsetted    |        |
| **Total** | **? KB**     | **< 500 KB** |        |

---

## Phase 3: Identify Hotspots

Scan for these common carbon offenders:

### 🔴 Critical (fix immediately)

| Hotspot                                  | How to detect                                         | Fix                                       |
| ---------------------------------------- | ----------------------------------------------------- | ----------------------------------------- |
| Unoptimised images (PNG/JPEG for photos) | `find dist -name "*.png" -size +100k`                 | Convert to AVIF/WebP                      |
| Missing lazy loading                     | Search for `<img` without `loading="lazy"`            | Add `loading="lazy"` to below-fold images |
| Render-blocking CSS/JS                   | Check `<head>` for non-deferred `<link>` / `<script>` | Defer or inline critical CSS              |
| Excessive client-side JS                 | `find dist -name "*.js" -exec du -ch {} +`            | Remove or lazy-load                       |

### 🟠 Important (fix soon)

| Hotspot                            | How to detect                             | Fix                                |
| ---------------------------------- | ----------------------------------------- | ---------------------------------- |
| No caching headers                 | Check `_headers` or server config         | Add immutable + revalidate headers |
| Third-party font requests          | Search for `fonts.googleapis.com` in HTML | Self-host via @fontsource          |
| Missing `width`/`height` on images | Grep `<img` without dimensions            | Add explicit dimensions            |
| No `prefers-reduced-motion`        | Search global CSS                         | Add motion reduction media query   |

### 🟡 Nice to have

| Hotspot                    | How to detect             | Fix                          |
| -------------------------- | ------------------------- | ---------------------------- |
| No `carbon.txt`            | Check `public/carbon.txt` | Add using template           |
| No `llms.txt`              | Check `public/llms.txt`   | Add structured site summary  |
| Missing JSON-LD            | Check page source         | Add structured data          |
| Large DOM (>1500 elements) | Browser DevTools          | Simplify component structure |

---

## Phase 4: Prioritise

Rank fixes by **bytes saved** (descending). The biggest files yield the most carbon reduction.

### Typical priority order

1. **Images** — usually 60-80% of page weight. Converting one large PNG to AVIF can save megabytes.
2. **Fonts** — removing unused weights or switching to system fonts saves 50-200 KB.
3. **JavaScript** — removing unused frameworks or lazy-loading saves 20-100 KB.
4. **CSS** — purging unused styles saves 5-30 KB.
5. **Caching** — doesn't reduce first-load weight but eliminates repeat transfers.
6. **HTML** — semantic cleanup has minimal byte impact but improves accessibility.

### Impact estimation

For each fix, estimate:

| Fix                      | Before     | After      | Savings             | Priority |
| ------------------------ | ---------- | ---------- | ------------------- | -------- |
| Convert hero.png to AVIF | 2.1 MB     | 85 KB      | 2.0 MB              | 🔴 P0    |
| Remove jQuery            | 89 KB      | 0 KB       | 89 KB               | 🟠 P1    |
| Self-host Google Fonts   | 3 requests | 0 requests | ~45 KB + latency    | 🟠 P1    |
| Add caching headers      | —          | —          | Repeat visits: 0 KB | 🟡 P2    |

---

## Phase 5: Refactor

Apply fixes **one category at a time** in priority order. After each fix:

1. Rebuild: `bun run build`
2. Re-measure: compare against Phase 2 baseline
3. Verify: run the [verification checklist](../assets/checklist.md)
4. Commit: document the carbon saving in the commit message

### Refactoring workflow

```
For each hotspot (in priority order):
  1. Read the relevant technique from carbon-techniques.md
  2. Apply the fix
  3. Build and measure
  4. Verify no regressions
  5. Commit with message: "perf: [description] — saves X KB"
  6. Move to next hotspot
```

### When to stop

Stop optimising when:

- Total page weight is under 500 KB (ideal: under 200 KB)
- JS ≤ 50 KB and CSS ≤ 20 KB per page
- All images are in modern formats with responsive srcset
- Caching headers are configured
- Further optimisation would degrade UX or maintainability

---

## Reporting

After completing the audit, generate a summary:

```markdown
## Carbon Audit Report — [Site Name]

### Before

- Total page weight: X KB
- JS: X KB | CSS: X KB | Images: X KB | Fonts: X KB

### After

- Total page weight: X KB (↓ X% reduction)
- JS: X KB | CSS: X KB | Images: X KB | Fonts: X KB

### Changes Applied

1. [Change description] — saved X KB
2. [Change description] — saved X KB
   …

### Remaining Opportunities

- [Low-priority items not yet addressed]
```
