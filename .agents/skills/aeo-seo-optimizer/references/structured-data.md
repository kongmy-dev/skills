# Structured Data (JSON-LD) Recipes

JSON-LD is the only structured data format Google, Bing, and most AI engines
prefer. Microdata and RDFa still work but have no advantages and bloat HTML.

**Always inline JSON-LD** in `<head>` — never load it from an external file.
External JSON-LD costs an extra request and is sometimes blocked by CSP.

**Always validate** with <https://validator.schema.org/> and
<https://search.google.com/test/rich-results>.

---

## 1. Organization (every site)

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Organization",
    "name": "Brand Name",
    "url": "https://example.com",
    "logo": "https://example.com/logo.png",
    "sameAs": [
      "https://linkedin.com/company/brand",
      "https://github.com/brand",
      "https://x.com/brand"
    ],
    "contactPoint": {
      "@type": "ContactPoint",
      "email": "hello@example.com",
      "contactType": "customer service"
    }
  }
</script>
```

Use `LocalBusiness` (or a more specific subtype like `ProfessionalService`)
**instead of** plain `Organization` if you have a physical location.

```jsonc
{
  "@context": "https://schema.org",
  "@type": ["ProfessionalService", "LocalBusiness"], // dual-type allowed
  "name": "…",
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "Kuala Lumpur",
    "addressRegion": "Kuala Lumpur",
    "addressCountry": "MY",
  },
  "geo": { "@type": "GeoCoordinates", "latitude": 3.121, "longitude": 101.73 },
  "openingHoursSpecification": [
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"],
      "opens": "09:00",
      "closes": "18:00",
    },
  ],
  "priceRange": "$$",
  "areaServed": [{ "@type": "Country", "name": "Malaysia" }],
}
```

---

## 2. WebSite + SearchAction (homepage only)

Crucial for AEO. Without it, Google can't render a sitelinks search box and
some AI engines miss your domain entirely.

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "WebSite",
    "name": "Brand",
    "url": "https://example.com",
    "potentialAction": {
      "@type": "SearchAction",
      "target": {
        "@type": "EntryPoint",
        "urlTemplate": "https://example.com/search?q={search_term_string}"
      },
      "query-input": "required name=search_term_string"
    }
  }
</script>
```

Omit `potentialAction` if your site has no search.

---

## 3. Article / BlogPosting

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "Title (≤ 110 chars)",
    "description": "Same as meta description",
    "image": ["https://example.com/og.png"],
    "datePublished": "2026-05-07T09:00:00+08:00",
    "dateModified": "2026-05-07T09:00:00+08:00",
    "author": {
      "@type": "Person",
      "name": "Author Name",
      "url": "https://example.com/authors/author-name"
    },
    "publisher": {
      "@type": "Organization",
      "name": "Brand",
      "logo": { "@type": "ImageObject", "url": "https://example.com/logo.png" }
    },
    "mainEntityOfPage": "https://example.com/article-slug"
  }
</script>
```

`dateModified` is what AI engines use to judge freshness — always update it.

---

## 4. FAQPage (powerful for AEO)

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "FAQPage",
    "mainEntity": [
      {
        "@type": "Question",
        "name": "What does KONGMY do?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "KONGMY is an independent technology consultancy that helps SMEs cut costs and automate operations through serverless infrastructure, AI workflows, and custom software."
        }
      },
      {
        "@type": "Question",
        "name": "Where are you located?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "Kuala Lumpur, Malaysia. We serve clients across Southeast Asia."
        }
      }
    ]
  }
</script>
```

**Rules:**

- Answer must be **verbatim** in the visible page content. Mismatch = warning in Google Search Console.
- Plain text only. Limited HTML allowed: `<a>`, `<b>`, `<em>`, `<i>`, `<strong>`, `<p>`, `<br>`.
- Keep answers ≤ 300 chars for AI surfaces; longer is fine for SERP.
- 5–10 questions per page is the sweet spot.

---

## 5. BreadcrumbList (every non-home page)

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    "itemListElement": [
      { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://example.com" },
      {
        "@type": "ListItem",
        "position": 2,
        "name": "Services",
        "item": "https://example.com/services"
      },
      {
        "@type": "ListItem",
        "position": 3,
        "name": "Web Hosting",
        "item": "https://example.com/services/web-hosting"
      }
    ]
  }
</script>
```

The visible breadcrumb on the page is optional but improves UX and CTR.

---

## 6. Service + OfferCatalog

For each service offered:

```jsonc
{
  "@context": "https://schema.org",
  "@type": "Service",
  "serviceType": "Website & App Hosting",
  "provider": { "@type": "Organization", "name": "Brand", "url": "https://example.com" },
  "areaServed": "Malaysia",
  "description": "Reliable, scalable hosting that doesn't require a dedicated IT person.",
  "offers": {
    "@type": "Offer",
    "priceCurrency": "MYR",
    "price": "0",
    "availability": "https://schema.org/InStock",
    "url": "https://example.com/services/hosting",
  },
}
```

For a list, use `OfferCatalog` on the parent Organization:

```jsonc
"hasOfferCatalog": {
  "@type": "OfferCatalog",
  "name": "Services",
  "itemListElement": [
    { "@type": "Offer", "itemOffered": { "@type": "Service", "name": "Hosting" } },
    { "@type": "Offer", "itemOffered": { "@type": "Service", "name": "AI Automation" } }
  ]
}
```

---

## 7. Person (founder / author bio)

```jsonc
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "Kong MY",
  "jobTitle": "Founder & Principal Consultant",
  "url": "https://kongmy.dev/about",
  "image": "https://kongmy.dev/about_me_kongmy.webp",
  "worksFor": { "@type": "Organization", "name": "KONGMY DIGITAL SOLUTIONS" },
  "sameAs": ["https://linkedin.com/in/kongmy", "https://github.com/kuribu99"],
}
```

---

## 8. Speakable (AEO bonus)

Marks short factual passages a voice assistant or AI summary can read aloud.

```jsonc
"speakable": {
  "@type": "SpeakableSpecification",
  "cssSelector": [".hero-headline", ".key-summary"]
}
```

Add it inside a `WebPage` or `Article` block. Pick selectors that contain
**self-contained, factual sentences** — not navigation or marketing fluff.

---

## 9. Combining with @graph

When a single page carries multiple types that reference each other, use a
`@graph` to keep IDs clean:

```jsonc
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "Organization",
      "@id": "https://example.com/#org",
      "name": "Brand",
      "url": "https://example.com",
    },
    {
      "@type": "WebSite",
      "@id": "https://example.com/#site",
      "url": "https://example.com",
      "publisher": { "@id": "https://example.com/#org" },
    },
    {
      "@type": "WebPage",
      "@id": "https://example.com/#home",
      "url": "https://example.com",
      "isPartOf": { "@id": "https://example.com/#site" },
    },
  ],
}
```

Stable `@id` values let crawlers connect entities across pages.

---

## 10. Common Pitfalls

| Pitfall                                     | Fix                                                                |
| ------------------------------------------- | ------------------------------------------------------------------ |
| Schema content not visible on the page      | Move content into rendered HTML; otherwise Google issues warnings. |
| `dateModified` stale or missing             | Generate at build from git or CMS metadata.                        |
| Wrong schema type (`Product` for service)   | Use `Service` for services, `Product` only for tangible goods.     |
| Missing `mainEntityOfPage` on Article       | Add the canonical URL.                                             |
| Multiple Organization blocks (one per page) | Define once on homepage, reference by `@id` elsewhere.             |
| Loading JSON-LD from external file          | Inline always; some crawlers ignore external.                      |
| Logo not square or > 600×600                | Provide a square logo ≥ 112×112 (Google's minimum).                |
| `priceRange` missing on LocalBusiness       | Add `"$"`, `"$$"`, `"$$$"`, or `"$$$$"`.                           |
