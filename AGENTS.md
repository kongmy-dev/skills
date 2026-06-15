# Agent Guidelines for KONGMY AI Skills

This repository is the central library of reusable AI agent skills for KONGMY Digital Solutions (`kongmy.dev`).

## Core Rules

- **Reusable Skills Only**: This repository is strictly for skills that are consumed across multiple projects. Project-specific skills or third-party vendor skills (like Firebase or PostHog) should remain in their respective project repositories.
- **Specification**: All skills MUST follow the [agentskills.io specification](https://agentskills.io/specification).
- **Naming**: Skill directories and their `name` field must match. Use lowercase alphanumeric characters and hyphens only (`a-z`, `0-9`, `-`). Do not start, end, or use consecutive hyphens. Maximum length is 64 characters.

## Creating a New Skill

When creating or modifying a skill, adhere to the following best practices derived from Anthropic's `skill-creator`:

1. **"Pushy" Descriptions**: The `description` field in the `SKILL.md` frontmatter is the primary triggering mechanism. It must explicitly state the contexts and user phrases when the skill should be used.
2. **Progressive Disclosure**: Agents load context progressively.
   - `SKILL.md` should be kept lean (<500 lines).
   - Place detailed technical documentation, formats, and schemas in the `references/` directory.
   - Place scripts in the `scripts/` directory.
   - Place templates in the `assets/` directory.
3. **Explain the "Why"**: Use theory of mind. Explain the rationale behind instructions instead of relying on rigid, unexplained `MUST` rules.
4. **Actionable Workflows**: Break complex workflows down into numbered, imperative steps.

## Cross-Repo Synchronization

When modifying a skill, ensure you communicate with the user regarding downstream updates. Downstream projects use git submodules with sparse checkout to consume these skills. Changes merged here will need to be pulled by downstream repositories.
