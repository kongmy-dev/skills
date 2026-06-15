# Astro SSG Patterns for Low-Carbon Sites

Astro-specific techniques extracted from production low-carbon websites.
Load this reference **only when generating or refactoring Astro projects**.

---

## 1. Project Configuration

### astro.config.mjs

```js
import { defineConfig } from 'astro/config';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://example.com', // Required for canonical URLs & sitemap

  // SSG: pre-render all pages at build time → zero server CPU at runtime
  output: 'static',

  integrations: [
    sitemap(), // Auto-generates sitemap XML at build time
  ],

  image: {
    // Generate modern formats automatically
    formats: ['avif', 'webp'],
  },
});
```

### Key rules

| Setting         | Value              | Why                                                |
| --------------- | ------------------ | -------------------------------------------------- |
| `output`        | `'static'`         | Pre-renders everything; zero runtime server cost   |
| `image.formats` | `['avif', 'webp']` | Smallest image files with broad browser support    |
| `site`          | Full URL           | Required for canonical links, OG tags, and sitemap |

### Adding Cloudflare adapter

When deploying to Cloudflare, add the adapter — see [cloudflare-optimization.md](cloudflare-optimization.md).

```js
import cloudflare from '@astrojs/cloudflare';

export default defineConfig({
  output: 'static',
  adapter: cloudflare({
    imageService: 'compile', // Build-time image optimisation (free)
  }),
});
```

---

## 2. Component Architecture

### Zero-JS by default

`.astro` components ship **no client-side JavaScript**. They render to HTML at build time.

```astro
---
/**
 * @component ServiceCard
 * @param {string} title - Service name
 * @param {string} description - Service description
 * @param {string} icon - Material icon name
 * @param {string} [class] - Additional CSS classes
 */
interface Props {
  title: string;
  description: string;
  icon: string;
  class?: string;
}

const { title, description, icon, class: className = '' } = Astro.props;
---

<article class={`service-card ${className}`}>
  <span class="material-symbols-outlined" aria-hidden="true">{icon}</span>
  <h3>{title}</h3>
  <p>{description}</p>
</article>

<style>
  .service-card {
    padding: 1.5rem;
    border: 1px solid var(--color-surface);
    border-radius: 0.5rem;
  }
</style>
```

### Extraction criteria

Extract into a separate `.astro` component when:

| Signal                      | Example                                |
| --------------------------- | -------------------------------------- |
| Block exceeds ~40 lines     | `HeroSection`, `AboutSection`          |
| Same structure in ≥2 places | `ServiceCard`, `LegalPageLayout`       |
| Block imports its own data  | `ServicesSection` → `data/services.ts` |
| Block contains `<script>`   | `HeroSection` (word rotator)           |
| Content is data-driven      | Technologies list, contact links       |

### Page files = table of contents

```astro
---
import Layout from '../layouts/Layout.astro';
import HeroSection from '../components/HeroSection.astro';
import ServicesSection from '../components/ServicesSection.astro';
import AboutSection from '../components/AboutSection.astro';
import ContactSection from '../components/ContactSection.astro';
---

<Layout title="Home | Example" description="Your trusted tech advisor.">
  <main>
    <HeroSection />
    <ServicesSection />
    <AboutSection />
    <ContactSection />
  </main>
</Layout>
```

---

## 3. Data-Driven Components

Drive variable lists from **typed data arrays** in `src/data/`:

```ts
// src/data/services.ts
export interface Service {
  title: string;
  description: string;
  icon: string;
}

export const services: readonly Service[] = [
  { title: 'Web Hosting', description: 'Reliable, scalable hosting.', icon: 'cloud' },
  { title: 'AI Automation', description: 'Smart workflow automation.', icon: 'smart_toy' },
] as const;
```

```astro
---
import { services } from '../data/services';
import ServiceCard from './ServiceCard.astro';
---

<section>
  {services.map((svc) => <ServiceCard {...svc} />)}
</section>
```

The template needs **zero changes** when items are added or removed.

---

## 4. Layout Pattern

### Root layout shell

```astro
---
import '../styles/global.css';
import SiteHeader from '../components/SiteHeader.astro';
import SiteFooter from '../components/SiteFooter.astro';

interface Props {
  title: string;
  description?: string;
  image?: string;
  canonicalUrl?: string;
}

const {
  title,
  description = 'Default meta description.',
  image = '/og-image.png',
  canonicalUrl,
} = Astro.props;

const siteUrl = Astro.site;
const canonical = canonicalUrl ?? new URL(Astro.url.pathname, siteUrl).href;
const ogImage = siteUrl ? new URL(image, siteUrl).href : image;
const hasHeadExtras = Astro.slots.has('head-extras');
---

<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{title}</title>
    <meta name="description" content={description} />
    <link rel="canonical" href={canonical} />

    <!-- Open Graph -->
    <meta property="og:title" content={title} />
    <meta property="og:description" content={description} />
    <meta property="og:image" content={ogImage} />

    <!-- Sitemap + LLMs discovery -->
    <link rel="sitemap" type="application/xml" href="/sitemap-index.xml" />
    <link rel="llms" href="/llms.txt" />

    {hasHeadExtras && <slot name="head-extras" />}
  </head>
  <body>
    <SiteHeader />
    <slot />
    <SiteFooter />
  </body>
</html>
```

### head-extras slot

Pages inject page-specific `<meta>`, `<link>`, or JSON-LD via the `head-extras` slot.
**Never modify Layout.astro for one-off page needs.**

```astro
<Layout title="Services | Brand">
  <script
    type="application/ld+json"
    slot="head-extras"
    set:html={JSON.stringify({
      '@context': 'https://schema.org',
      '@type': 'Service',
    })}
  />
  <main>…</main>
</Layout>
```

---

## 5. Image Pipeline

### Using Astro's built-in `<Image>` component

```astro
---
import { Image } from 'astro:assets';
import heroImg from '../assets/hero.jpg';
---

<Image
  src={heroImg}
  alt="Hero image"
  widths={[400, 800, 1200]}
  sizes="(max-width: 768px) 100vw, 800px"
  loading="lazy"
  format="avif"
/>
```

### Rules

- LCP images: add `fetchpriority="high"` and `loading="eager"`
- Below-fold images: always `loading="lazy"`
- Set `formats: ['avif', 'webp']` in `astro.config.mjs`
- Use `imageService: 'compile'` with Cloudflare adapter for free build-time optimisation

---

## 6. Font Loading

### Self-hosted via @fontsource

Prefer **variable** packages — one file covers the entire weight axis:

```astro
---
// Layout.astro frontmatter
import '@fontsource-variable/inter';
---
```

If you must use static weights, import only the ones used. Audit before
shipping:

```bash
# Show every font-weight value referenced in source
rg -o 'font-(thin|extralight|light|normal|medium|semibold|bold|extrabold|black)' src
```

Drop any `@fontsource/<name>/<weight>.css` import that doesn't match the
output. Common mistake: importing 3, 4, or 5 weights when the page actually
uses 2.

### Deferred icon fonts

```html
<!-- Non-critical icon font: defer with media=print trick -->
<link
  rel="stylesheet"
  href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined"
  media="print"
  onload="this.media='all'"
/>
<noscript>
  <link
    rel="stylesheet"
    href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined"
  />
</noscript>
```

---

## 7. Styling

### Design tokens in global CSS

```css
@import 'tailwindcss';

@theme {
  --color-primary: #0a192f;
  --color-accent: #c5a065;
  --color-surface: #f4f6f8;
  --font-serif: 'Newsreader', serif;
  --font-sans: 'Source Sans 3', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
}

html {
  scroll-behavior: smooth;
}
body {
  font-family: var(--font-sans);
}
h1,
h2,
h3,
h4,
h5,
h6 {
  font-family: var(--font-serif);
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### Scoped component styles

Astro automatically scopes `<style>` blocks to the component. This prevents CSS bloat from global stylesheets.

---

## 8. Client-Side JS Hierarchy

From most to least preferred:

| #   | Approach            | When                               |
| --- | ------------------- | ---------------------------------- |
| 1   | No JS               | Plain HTML/CSS solution            |
| 2   | Vanilla `<script>`  | Native browser APIs, single file   |
| 3   | Web Components      | Encapsulated, framework-agnostic   |
| 4   | `client:idle`       | Hydrate after browser idle         |
| 5   | `client:only`       | Hydrate on load (avoid)            |
| 6   | Tiny library ≤ 5 KB | Well-maintained, gzipped           |
| 7   | Larger library      | Exceptional justification required |

When adding JS, document:

- Runtime size (gzipped)
- Alternatives considered
- Why CSS/HTML cannot achieve the same result
- Graceful degradation for non-JS users
