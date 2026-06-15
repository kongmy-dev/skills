# Cloudflare Deployment Optimisation

Cloudflare-specific configurations for minimising carbon footprint at the edge.
Load this reference **only when deploying to Cloudflare Pages/Workers**.

---

## 1. Astro Adapter Configuration

```js
// astro.config.mjs
import cloudflare from '@astrojs/cloudflare';

export default defineConfig({
  output: 'static',
  adapter: cloudflare({
    // Build-time image optimisation via sharp — free, no Cloudflare Image Resizing needed
    imageService: 'compile',
  }),
});
```

### Why `imageService: 'compile'`?

- Processes images at **build time** using sharp
- No runtime compute or paid Cloudflare Image Resizing service
- Generates AVIF + WebP variants that are served statically from the CDN

---

## 2. Wrangler Configuration

```jsonc
// wrangler.jsonc
{
  "name": "my-site",
  // Worker entry emitted by `astro build`
  "main": "dist/_worker.js/index.js",
  // Use the latest compatibility date for smallest polyfill surface
  "compatibility_date": "2026-02-25",
  // global_fetch_strictly_public: blocks requests to private/loopback IPs
  // Zero runtime cost, good security default
  // nodejs_compat intentionally omitted: SSG pages don't execute Node.js APIs
  // Removing this keeps Worker bundle smaller (no Node.js polyfills)
  "compatibility_flags": ["global_fetch_strictly_public"],
  "assets": {
    "binding": "ASSETS",
    "directory": "./dist",
    "html_handling": "auto-trailing-slash",
    "not_found_handling": "404-page",
  },
  // Zero-cost observability (included on all plans)
  "observability": {
    "logs": { "enabled": true, "invocation_logs": true },
    "traces": { "enabled": true },
  },
}
```

### Key optimisation flags

| Flag                                     | Effect                     | Carbon Impact                 |
| ---------------------------------------- | -------------------------- | ----------------------------- |
| `global_fetch_strictly_public`           | Prevents SSRF              | Zero runtime cost             |
| Omit `nodejs_compat`                     | No Node.js polyfill bundle | Smaller Worker bundle         |
| `"not_found_handling": "404-page"`       | Serves static 404          | No Worker invocation for 404s |
| `"html_handling": "auto-trailing-slash"` | Clean URLs                 | No redirect round-trips       |

---

## 3. Static Asset Caching via `_headers`

Place `public/_headers` for Cloudflare Pages:

```
# Security headers — zero cost, every response
/*
  X-Frame-Options: SAMEORIGIN
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()

# Hashed Astro build assets — cache forever
/_astro/*
  Cache-Control: public, max-age=31536000, immutable

# Static brand/favicon assets
/favicon.ico
  Cache-Control: public, max-age=31536000, immutable
/favicon-16x16.png
  Cache-Control: public, max-age=31536000, immutable
/favicon-32x32.png
  Cache-Control: public, max-age=31536000, immutable
/apple-touch-icon.png
  Cache-Control: public, max-age=31536000, immutable

# HTML pages — always check for freshness
/*.html
  Cache-Control: public, max-age=0, must-revalidate
```

### Why this matters

- **Immutable hashed assets**: Cloudflare's CDN caches these at every edge PoP. Repeat visitors transfer **zero bytes** for CSS/JS/images.
- **HTML revalidation**: Ensures content updates propagate instantly without purging caches.
- **Edge-served**: Static assets are served directly from Cloudflare's CDN. The Worker is only invoked for dynamic routes.

---

## 4. `.assetsignore`

Exclude development-only files from the CDN deployment:

```
# public/.assetsignore
_worker.js
```

This prevents Cloudflare from serving internal build artifacts as static assets.

---

## 5. Edge Architecture

```
User Request
    │
    ▼
┌─────────────────────┐
│  Cloudflare Edge PoP │  ← Nearest to user's location
│  (CDN Cache Layer)   │
└──────────┬──────────┘
           │
     Cache HIT?
    ┌──────┴──────┐
    │ Yes         │ No
    ▼             ▼
  Response    ┌─────────┐
  (0 compute) │ Worker  │ ← Only for dynamic routes
              └────┬────┘
                   │
                Response
                (cached for next request)
```

### Carbon implications

- **SSG + CDN = no origin server**: Pages are pre-built and distributed globally
- **Worker invocations are rare**: with `output: 'static'`, the Worker handles only programmatic routes (typically none)
- **Geographic proximity**: Cloudflare has 300+ PoPs; content is served from the nearest one

---

## 6. carbon.txt Upstream Declaration

When using Cloudflare, declare it in your `carbon.txt`:

```toml
[upstream]
services = [
  { domain = "cloudflare.com", service_type = ["cdn", "shared-hosting"] }
]
```

---

## 7. Performance Monitoring (Free)

### Workers Logs

Enabled via `observability.logs` in wrangler config. Zero additional cost.

### Workers Traces

Enabled via `observability.traces`. Provides request tracing without additional services.

### Cloudflare Analytics

- **Web Analytics** (free): Client-side, privacy-respecting analytics
- **Workers Analytics** (free): Request counts, CPU time, errors

Use these to identify pages with unexpectedly high transfer sizes or error rates.
