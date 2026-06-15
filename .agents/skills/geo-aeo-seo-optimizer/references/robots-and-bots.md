# robots.txt and AI Bot Policy

`robots.txt` is the front door for crawlers — both classic search bots and
the new generation of AI training/retrieval bots. A site without an explicit
AI policy implicitly opts in to **everyone**.

---

## 1. Minimum Viable robots.txt

```
User-agent: *
Allow: /

Sitemap: https://example.com/sitemap-index.xml
```

That's the floor. It tells classic crawlers everything is fair game and
points them at the sitemap.

---

## 2. The AI Crawler Matrix

Two distinct purposes, often conflated:

| Purpose       | What the bot does                               | Examples                                                                                              |
| ------------- | ----------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Training**  | Crawls content to train future model versions.  | `GPTBot`, `Google-Extended`, `CCBot`, `anthropic-ai`, `Bytespider`, `Amazonbot`                       |
| **Retrieval** | Fetches pages on-demand to answer a user query. | `OAI-SearchBot`, `ChatGPT-User`, `ClaudeBot`, `PerplexityBot`, `Perplexity-User`, `Applebot-Extended` |

**Key distinction**: blocking _training_ bots stops your content from
becoming model knowledge. Blocking _retrieval_ bots stops you from being
cited in answers. Most marketing sites want retrieval allowed even if
training is blocked.

### Current bot user-agent strings (verify on each vendor's docs)

| User-agent             | Operator     | Purpose                       |
| ---------------------- | ------------ | ----------------------------- |
| `GPTBot`               | OpenAI       | Training                      |
| `OAI-SearchBot`        | OpenAI       | Retrieval (ChatGPT Search)    |
| `ChatGPT-User`         | OpenAI       | User-triggered fetches        |
| `Google-Extended`      | Google       | Training (Gemini, Vertex AI)  |
| `Googlebot`            | Google       | Classic search index          |
| `ClaudeBot`            | Anthropic    | Training                      |
| `anthropic-ai`         | Anthropic    | Legacy training crawler       |
| `Claude-Web`           | Anthropic    | Retrieval (Claude with web)   |
| `PerplexityBot`        | Perplexity   | Index for Perplexity search   |
| `Perplexity-User`      | Perplexity   | User-triggered fetches        |
| `CCBot`                | Common Crawl | Open dataset (used by many)   |
| `Bytespider`           | ByteDance    | Training (TikTok / Doubao)    |
| `Amazonbot`            | Amazon       | Training (Alexa / Q)          |
| `Applebot-Extended`    | Apple        | Training (Apple Intelligence) |
| `Applebot`             | Apple        | Classic Siri/Spotlight        |
| `Meta-ExternalAgent`   | Meta         | Training                      |
| `Meta-ExternalFetcher` | Meta         | Retrieval                     |
| `cohere-ai`            | Cohere       | Training                      |
| `DuckAssistBot`        | DuckDuckGo   | Retrieval                     |
| `MistralAI-User`       | Mistral      | Retrieval                     |

The list grows; check <https://github.com/ai-robots-txt/ai.robots.txt> for an
updated registry.

---

## 3. Recipe: Allow Everything (default for most marketing sites)

```
User-agent: *
Allow: /

# AI policy: explicit allow (default behaviour, but documents intent)
User-agent: GPTBot
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Google-Extended
Allow: /

Sitemap: https://example.com/sitemap-index.xml
```

Why explicit? Future crawlers may default to "ask for permission". Listing
intent makes the policy machine-readable.

---

## 4. Recipe: Allow Retrieval, Block Training

For sites that want to be **cited** in AI answers but not **memorised** by
model training. Common for premium content, news, paywalled-adjacent sites.

```
User-agent: *
Allow: /

# Block training crawlers
User-agent: GPTBot
Disallow: /

User-agent: ClaudeBot
Disallow: /

User-agent: anthropic-ai
Disallow: /

User-agent: Google-Extended
Disallow: /

User-agent: CCBot
Disallow: /

User-agent: Bytespider
Disallow: /

User-agent: Amazonbot
Disallow: /

User-agent: Applebot-Extended
Disallow: /

User-agent: Meta-ExternalAgent
Disallow: /

# Allow retrieval crawlers (used for live citations)
User-agent: OAI-SearchBot
Allow: /

User-agent: ChatGPT-User
Allow: /

User-agent: Claude-Web
Allow: /

User-agent: Perplexity-User
Allow: /

User-agent: Meta-ExternalFetcher
Allow: /

Sitemap: https://example.com/sitemap-index.xml
```

---

## 5. Recipe: Block Everything (worst-case for AEO)

Don't do this on a site you want discovered. Included only for completeness:

```
User-agent: GPTBot
User-agent: ClaudeBot
User-agent: PerplexityBot
User-agent: Google-Extended
User-agent: CCBot
User-agent: Bytespider
User-agent: Amazonbot
User-agent: Applebot-Extended
User-agent: anthropic-ai
User-agent: Meta-ExternalAgent
Disallow: /
```

Note: blocking does **not** retroactively remove your content from already-trained
models. It only affects future crawls.

---

## 6. Targeted Disallows

Hide low-value pages even from classic crawlers:

```
User-agent: *
Disallow: /admin/
Disallow: /api/
Disallow: /search?
Disallow: /*.json$
Disallow: /thank-you
Disallow: /staging/

Sitemap: https://example.com/sitemap-index.xml
```

Always include the sitemap line. Multiple `Sitemap:` lines are allowed.

---

## 7. HTTP-Header Alternative: `X-Robots-Tag`

For non-HTML resources or per-route control:

```
# Cloudflare _headers file
/api/*
  X-Robots-Tag: noindex, nofollow

/staging/*
  X-Robots-Tag: noindex, nofollow

/*.pdf
  X-Robots-Tag: noindex
```

Useful when:

- The asset is not HTML (PDF, image, JSON).
- You can't easily add `<meta name="robots">` to a generated page.

---

## 8. Validation

- Google Search Console → "robots.txt Tester".
- <https://www.opensiteexplorer.org/robots-txt-tester>
- Bot-specific: vendors usually publish a fetcher you can test against.

After deploy, **verify that `/robots.txt` returns 200 with `Content-Type:
text/plain`**. A surprising number of sites accidentally serve HTML 404s.

---

## 9. Related Files

- `/sitemap-index.xml` — declared in robots.txt.
- `/llms.txt` — referenced via `LLMs:` directive (informal).
- `/security.txt` — `.well-known/security.txt`, separate spec, vulnerability disclosure.
- `/ai.txt` — proposed by Spawning, less adopted than llms.txt.
- `/carbon.txt` — sustainability disclosure (see low-carbon-web-generator skill).
