# Third-Party Weight — Common Offenders and How to Cut Them

Third-party scripts and embeds are the largest carbon offenders on most
otherwise-lean sites. A single Google Maps iframe or duplicate analytics
tag can outweigh the entire first-party bundle.

This reference catalogues the offenders we hit most often and the
"keep-the-feature, cut-the-bytes" pattern for each.

---

## 1. Map Embeds (Google Maps, Mapbox, Leaflet)

### Cost (Google Maps iframe)

| Resource           | Size        |
| ------------------ | ----------- |
| Map JS bundle      | ~400–600 KB |
| Tile images        | 50–200 KB   |
| Fonts + CSS        | ~50 KB      |
| **Total per page** | **~700 KB** |

Even with `loading="lazy"`, the iframe is fetched as soon as it scrolls
near the viewport — and on short pages it loads immediately.

### Pattern: Static Map + Click to Open

Replace the iframe with a static screenshot that links to the live map.

```astro
---
const lat = 3.121625;
const lng = 101.730683;
const mapUrl = `https://www.google.com/maps/place/?q=place_id:CHIJ…`;
---

<a href={mapUrl} target="_blank" rel="noopener noreferrer" aria-label="Open in Google Maps">
  <Image
    src={import('../assets/office-map.png')}
    alt="KONGMY DIGITAL SOLUTIONS office location, Kuala Lumpur"
    width={600}
    height={300}
    loading="lazy"
  />
</a>
```

Generate the screenshot once at design time (Maps Static API, OpenStreetMap
export, or a manual screenshot). Re-export only when the location changes.

### Pattern: Click-to-Load Embed

If users genuinely need to interact (zoom, drag), defer the iframe behind
an explicit click:

```astro
<button type="button" data-load-map class="map-placeholder" aria-label="Load interactive map">
  <img src="/static-map.webp" alt="Map preview" width="600" height="300" />
  <span>Click to load interactive map</span>
</button>

<script>
  document.querySelector('[data-load-map]')?.addEventListener('click', (e) => {
    const btn = e.currentTarget as HTMLButtonElement;
    const iframe = document.createElement('iframe');
    iframe.src = '…';
    iframe.title = 'Office location';
    iframe.loading = 'lazy';
    btn.replaceWith(iframe);
  });
</script>
```

Saves ~700 KB per page load for users who never click.

---

## 2. Analytics — One is Enough

Running multiple analytics platforms is the most common audit finding.

### Cost matrix

| Platform                 | Bundle | Notes                                     |
| ------------------------ | ------ | ----------------------------------------- |
| Google Analytics (gtag)  | ~50 KB | + first-party `dataLayer`                 |
| Google Tag Manager       | ~80 KB | Loads other scripts dynamically           |
| PostHog (full)           | ~95 KB | Includes session replay, feature flags    |
| PostHog (lite)           | ~30 KB | `opt_in_site_apps: false`, no replay      |
| Plausible                | ~1 KB  | Privacy-respecting, server-side filtering |
| Cloudflare Web Analytics | ~5 KB  | Free, RUM-based, server-aggregated        |
| Mixpanel                 | ~70 KB | Heavy product analytics                   |

### Rule

**Pick one** that matches your need:

- Marketing dashboards + GSC integration → Google Analytics
- Product analytics, feature flags → PostHog (lite mode)
- Privacy-first marketing → Plausible or Cloudflare Web Analytics
- Edge perf / RUM only → Cloudflare Web Analytics

If you genuinely need both marketing and product analytics, document the
~150 KB cost and consider deferring the product analytics until interaction
(e.g. user signs in / clicks CTA).

### Loading rule

Analytics is **never** above-the-fold critical path. Always `defer` or
`async`, ideally inject after the page loads:

```html
<script>
  window.addEventListener('load', () => {
    const s = document.createElement('script');
    s.src = 'https://cdn.example.com/analytics.js';
    s.async = true;
    document.head.appendChild(s);
  });
</script>
```

---

## 3. Social Embeds (Twitter, Instagram, YouTube)

### YouTube embed cost

| Resource      | Size     |
| ------------- | -------- |
| Player iframe | ~600 KB+ |
| Thumbnail     | ~30 KB   |
| Cookies       | Tracking |

### Pattern: `lite-youtube` or facade

Use a tiny wrapper that loads the full player only on click:

```html
<lite-youtube videoid="dQw4w9WgXcQ" playlabel="Play: Title"></lite-youtube>
```

`lite-youtube` ([github.com/paulirish/lite-youtube-embed](https://github.com/paulirish/lite-youtube-embed))
is ~3 KB. The full player loads on first click.

For Twitter/X: use a quoted blockquote with link, not the embed widget
(`platform.twitter.com/widgets.js` is ~120 KB).

For Instagram: use `og:image` thumbnail + link rather than the embed.

---

## 4. Icon Fonts vs Inline SVG

### Cost (Material Symbols / Font Awesome)

| Approach                       | Cost per page                                  |
| ------------------------------ | ---------------------------------------------- |
| Material Symbols Outlined font | ~150 KB woff2 (full set), even if 5 icons used |
| Material Symbols Variable      | ~330 KB                                        |
| Inline SVG (per icon)          | 0.5–2 KB each, only for icons actually used    |

### Rule

Switch to **inline SVG** when:

- You use ≤ 15 icons per page
- Icons are static (no dynamic icon swapping)
- You don't need ligature-based icon names

Keep the icon font when:

- You use 50+ unique icons across the site
- Icons are user-selectable from a large set
- You've subsetted the font (≤ 30 KB)

### Migration pattern

```astro
---
import iconArchitecture from '../assets/icons/architecture.svg?raw';
---

<span class="icon" aria-hidden="true" set:html={iconArchitecture} />
```

Or use Astro's `astro-icon` integration for build-time SVG inlining.

---

## 5. Carbon / Performance Badges

Badges that score your site (Website Carbon Badge, EcoGrader, etc.) make
**HTTP requests to a third-party API on every page load**. The irony is
that a sustainability badge often _increases_ the page's footprint.

### Pattern: Static badge image

Score your site once, screenshot the badge, publish as a static image:

```html
<a href="https://www.websitecarbon.com/website/example-com/" target="_blank">
  <img
    src="/wcb-badge.svg"
    alt="Website Carbon: A grade, cleaner than 90% of pages tested"
    width="120"
    height="48"
    loading="lazy"
  />
</a>
```

Re-score quarterly or after major changes; update the static asset.

---

## 6. Chat Widgets (Intercom, Drift, HubSpot, Tawk)

These are the second-largest weight after maps. Bundles routinely exceed
**1 MB**, and they connect to a WebSocket immediately.

### Pattern: Click-to-load chat

```html
<button id="open-chat" type="button">Chat with us</button>
<script>
  document.getElementById('open-chat')?.addEventListener('click', () => {
    const s = document.createElement('script');
    s.src = 'https://widget.intercom.io/widget/APP_ID';
    s.async = true;
    document.head.appendChild(s);
  });
</script>
```

Or replace the chat widget entirely with a WhatsApp/Telegram link — zero
client JS, native deep-links into the user's existing app.

---

## 7. Fonts — The Hidden Multiplier

`@fontsource` and Google Fonts make it easy to add fonts. Too easy.

### Common over-import

```ts
// 9 font files = ~270 KB
import '@fontsource/jetbrains-mono/400.css';
import '@fontsource/newsreader/400.css';
import '@fontsource/newsreader/400-italic.css';
import '@fontsource/newsreader/600.css';
import '@fontsource/newsreader/700.css';
import '@fontsource/source-sans-3/300.css';
import '@fontsource/source-sans-3/400.css';
import '@fontsource/source-sans-3/500.css';
import '@fontsource/source-sans-3/600.css';
```

### Rule

Audit which weights are actually used:

```bash
# Find every font-weight used in CSS/Tailwind classes
rg -o 'font-(thin|extralight|light|normal|medium|semibold|bold|extrabold|black)' src
```

Most sites need **3 weights total**: regular, semibold (or bold), italic.
Drop unused weights. Each removed file saves ~30 KB.

### Variable fonts

When a font ships a variable version, prefer it: one file covers all
weights, typically smaller than 2–3 static weights combined.

```ts
// Instead of 4 static files…
import '@fontsource-variable/inter';
// One ~50 KB file covers weight 100–900
```

---

## 8. CSS Frameworks

Tailwind v4 with proper purging is fine (typically < 20 KB). Other frameworks
to be wary of:

- **Bootstrap** full build: ~250 KB CSS + ~80 KB JS — almost always overkill.
- **Material UI / MUI**: 300 KB+ JS for the component runtime alone.
- **Bulma / Foundation**: similar issues without strong purging.

For static marketing sites, Tailwind utility-first or hand-written component
CSS beats every component framework on weight.

---

## 9. Quick Reference — Cut Order

When a site is over budget, attack in this order (largest savings first):

1. **Map iframes** (~700 KB each)
2. **Chat widgets** (~1 MB each)
3. **Video / social embeds** (~600 KB each)
4. **Duplicate analytics** (~80 KB each)
5. **Heavy icon fonts** (~150 KB)
6. **Excess font weights** (~30 KB each)
7. **Carbon / score badges** (~5–20 KB + extra requests)
8. **CSS framework over-purge** (~30–100 KB)
