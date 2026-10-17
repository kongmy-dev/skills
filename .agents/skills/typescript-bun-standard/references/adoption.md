# Adopting the standard in an existing repo

A standard that lands as one big reformat-and-refactor commit breaks things and
buries every real diff for months. Adopt it in this order.

1. **Toolchain first.** Pin `packageManager`, delete any other lockfile, commit
   `bun.lock`, switch CI to `bun install --frozen-lockfile`. Low risk, immediate
   payoff.
2. **Census before you configure.** Count what the tree already does (quote style,
   semicolons, line-length p95, how many `any`, `!` and `enum` there are) and write
   the config to match the majority. A formatter run that changes every file is a
   diff nobody can review.
3. **Compiler flags one at a time.** Turn on each `tsconfig` flag from
   `tsconfig.md`, fix what it reports, commit. A flag with too many findings to fix
   in one change waits; note it in the backlog instead of landing it half-fixed.
4. **Lint rules as errors, not warnings.** A warning is a rule nobody follows.
   Where a rule has a large existing count, relax it with a scoped override that
   names the reason, then shrink the override.
5. **Hooks.** Add `.githooks/pre-commit` and `pre-push` from `assets/githooks/`,
   wire them with the `prepare` script.
6. **Dead code.** Add `knip`; fix or justify every finding. Each exclusion in its
   config says why on the line above it.
7. **Gates with a ratchet.** A size gate or comment gate starts at the repo's own
   p95 and measures only staged files, so touching an old file brings its old
   findings and they're paid down by real splits. Never raise a limit.
8. **Boundaries last.** Encode each layer rule with `noRestrictedImports` once the
   layers exist. A boundary rule that already has fifty exceptions isn't a rule.

## Project rules on top

This standard is generic. Each project's own `CLAUDE.md`/`AGENTS.md` adds what's
specific to it: tenancy, money representation, layer names, the decision record,
the runtime per deployable. Put those there, not here; and when a project rule
contradicts this one, the project wins, and the difference is worth raising.

## Consuming the skill

See the repository README: add this repo as a submodule under
`.agents/shared-skills` with sparse checkout of
`.agents/skills/typescript-bun-standard`, then point the project's `CLAUDE.md` at
it ("Load the `typescript-bun-standard` skill before writing any `.ts`/`.tsx`").
