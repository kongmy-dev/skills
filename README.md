# kongmy.dev AI Skills

Central library of reusable AI agent skills for KONGMY Digital Solutions projects.

This repository stores highly reusable, KONGMY Digital Solutions-authored AI skills designed to be consumed by multiple downstream projects.

## Skill Format

All skills in this repository follow the [agentskills.io specification](https://agentskills.io/specification). 

Additionally, we adhere to the best practices enforced by the Anthropic `skill-creator`:
- **"Pushy" Descriptions**: Skills define explicitly when they should trigger using targeted keywords (e.g., `"Use when the user asks about..."`).
- **Progressive Disclosure**: `SKILL.md` is kept lean (under 500 lines). Detailed documentation, prompts, and templates are offloaded to a `references/` directory so agents only load context when required.
- **Explaining the "Why"**: Instructions prioritize explaining the rationale behind rules instead of using rigid `MUST` constraints.
- **Imperative Steps**: Workflows are broken down into clear, actionable steps.

## How to Consume Skills

Downstream projects consume these skills selectively using **git submodules** and **sparse checkout**. This keeps the project working tree clean by only downloading the skills the project actually needs.

Run the following commands in your downstream project root:

```bash
# 1. Add the skills repo as a submodule
git submodule add https://github.com/kongmy-dev/skills.git .agents/shared-skills

# 2. Enable sparse checkout inside the submodule
cd .agents/shared-skills
git sparse-checkout init --cone

# 3. Select only the skills you need
git sparse-checkout set .agents/skills/aeo-seo-optimizer .agents/skills/low-carbon-web-generator

# 4. Return to project root
cd ../..
```

## Commonly Used Skills Reference

While this repository hosts kongmy.dev authored reusable skills, our ecosystem also relies heavily on vendor-specific and third-party skills. To prevent breaking plugin/SDK update cycles, those third-party skills are managed in their respective project repositories.

### KONGMY Digital Solutions Skills

These are custom skills authored by kongmy.dev to standardize workflows across our projects.

| Skill | Source | Location | Notes |
|---|---|---|---|
| `aeo-seo-optimizer` | kongmy.dev | This repo (`.agents/skills/`) | AEO/SEO optimization |
| `low-carbon-web-generator` | kongmy.dev | This repo (`.agents/skills/`) | Green web development |

### Third-Party & Vendor Skills

These skills are provided by external vendors and frameworks, and are typically managed locally within the projects that use them.

| Skill | Vendor | Notes |
|---|---|---|
| `firebase-basics` | Firebase | Official Firebase CLI setup |
| `firebase-crashlytics` | Firebase | Crashlytics SDK setup |
| `integration-android` | PostHog | PostHog analytics for Android |
| `integration-astro-static` | PostHog | PostHog analytics for static Astro |
| `posthog-integration-astro-hybrid` | PostHog | PostHog for hybrid Astro |
| `frontend-design` | Anthropic | Production-grade UI design guidance |
| `web-design-guidelines` | Vercel | Web Interface Guidelines review |
| `skill-creator` | Anthropic | Used for drafting/improving new skills |
