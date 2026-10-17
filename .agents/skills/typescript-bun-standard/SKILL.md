---
name: typescript-bun-standard
description: >-
  The house standard for TypeScript projects built with Bun: toolchain, tsconfig
  strictness, Biome lint and format, module boundaries, types, errors and edges,
  env and config, tests, size and comment gates, git hooks and commits. Each rule
  names the check that catches a breach. Use before writing or changing any .ts or
  .tsx file in a Bun project, when scaffolding a new TypeScript/Bun repo or
  package, when adding tsconfig, biome.json, bunfig.toml or git hooks, when
  reviewing a TypeScript diff, or when the user asks for "coding standards",
  "coding principles", "lint rules", "strict TypeScript", "project conventions"
  or how code should be written in a Bun/TypeScript codebase.
license: MIT
compatibility: Bun 1.3+, TypeScript 6 (strict by default), Biome 2.5+. Works in Claude Code, Antigravity and other agentic coding environments.
metadata:
  author: kongmy
  version: '1.0'
---

# TypeScript/Bun standard

How TypeScript is written in a Bun project. A project's own `CLAUDE.md`/`AGENTS.md`
and its own `coding-principles` skill sit on top of this one: they add domain rules
(tenancy, money representation, layers) and they win where they differ. When you
find a difference, say so; don't silently pick one.

Every rule has an ID and a *Caught by*: **types** (the compiler), **lint** (Biome),
**gate** (a size or comment gate), **test**, **hook** (pre-commit or pre-push),
**review** (a reader, nothing else). A rule held only by review is a rule waiting
for a check; see "Changing the rules".

Older files break rules this standard sets. Copy the rule, not the neighbouring code.

## Before you write

1. **Decide out loud.** List what the request leaves open (input shape, output
   shape, edge behaviour), take the narrowest answer, say what you are *not*
   building, then build that list and stop. Unconstrained work adds fields and
   helpers nobody asked for; naming the boundary first is what stops it.
2. **Find the existing one.** A helper, schema, formatter or rule that already
   exists is used, not written again. Two implementations of one thing is a finding.
3. **Name the data first**, from how the caller uses it; then the type; then the code.

## Toolchain — `references/toolchain.md`

- **B1 · Bun only.** `bun install`, `bun run`, `bun test`, `bunx`, `bun add -g`.
  Never npm, npx, yarn or pnpm: one lockfile (`bun.lock`, committed), one resolver,
  one cache. Bun is the toolchain; the production runtime (Bun or Node) is a
  project decision written in its `CLAUDE.md`. *Caught by:* review; CI installs
  with `bun install --frozen-lockfile`.
- **B2 · Pin what writes config.** `packageManager: "bun@x.y.z"` in the root
  `package.json`; exact versions for TypeScript, Biome and Turbo, whose schemas and
  diagnostics change between minors. *Caught by:* review.
- **B3 · Read the docs before an unfamiliar API.** `ctx7 library <name> "<q>"`,
  then `ctx7 docs <id> "<q>"`, before writing against a new library, an evolving
  framework, a CLI or a migration. Trial-and-error against a remembered API ships
  the API as it was a year ago. *Caught by:* review.

## Compiler — `references/tsconfig.md`, `assets/tsconfig.base.json`

- **TS1 · Strict, plus the flags strict leaves off.** `strict`,
  `noUncheckedIndexedAccess`, `noImplicitOverride`, `noFallthroughCasesInSwitch`,
  `verbatimModuleSyntax`, `erasableSyntaxOnly`; `exactOptionalPropertyTypes` on new
  repos. One base config, extended by every package. *Caught by:* types.
- **TS2 · `types: ["bun"]` is explicit.** TypeScript 6 defaults `types` to `[]`, so
  a config without it loses Bun's globals. *Caught by:* types.
- **TS3 · Every file is type-checked by something.** Root scripts outside any
  workspace get their own `tsconfig.json` in the root `typecheck` script. *Caught
  by:* review.

## Lint and format — `references/biome.md`, `assets/biome.json`

- **L1 · Biome is the linter and the formatter**, one pass, `--error-on-warnings`.
  No ESLint or Prettier beside it. Formatting is never discussed in review: the
  tool owns it. *Caught by:* lint, hook.
- **L2 · Settle style once.** A new repo takes `assets/biome.json`. An existing repo
  censuses its tree (quote style, semicolons, line length p95) and adopts the
  majority, because that is the smallest diff that buys consistency. *Caught by:* lint.
- **L3 · No suppressions.** No `biome-ignore`, `@ts-ignore` or `@ts-expect-error`
  in shipped code. Write code the tool accepts; if none exists, ask. *Caught by:*
  review, comment gate where one exists.

## Shape of every file — `references/shape-and-comments.md`

- **S1 · No comments.** Names, types, structure and tests say *what*; the *why*
  goes to a decision record, a test name or the commit. Comments drift and nothing
  checks them; a paragraph explaining when two fields combine is a type that is too
  loose. Fixed exceptions only: a one-line foreign constraint with a link
  (`// vendor: <url> …`), a bare decision ID, a licence header. Existing comments
  are ratcheted: when you rewrite the code around one, move what it knew, then
  delete it. Never sweep. *Caught by:* gate where the repo has one, else review.
- **S2 · Write to the gate's shape.** Short functions with few decisions: guard
  clauses, `Record<Union, X>` tables instead of branches, named predicates, one job
  per function. Limits come from the repo's own p95, never copied from elsewhere,
  never raised to bless a file. *Caught by:* gate.
- **S3 · Subtract before you add.** Every new export has a consumer in the same
  change. No interface with one implementation, factory that builds one thing,
  options object read in one place, re-export shim or one-caller wrapper. The seam
  appears when the second caller does. *Caught by:* `knip` in CI, review.
- **S4 · Names carry meaning.** Files kebab-case (components may be PascalCase
  where the framework expects it); booleans `is`/`has`/`can`; a unit on every bare
  number (`timeoutMs`, `amountMinor`); `At` for an instant, `On` for a date. No
  `utils`, `helpers`, `misc`, `common`, `manager`, `data`, `info`. Named exports
  only, default exports where a framework requires them. More than two parameters,
  or two of one type, become one object. *Caught by:* review.

## Types — `references/types-errors-edges.md`

- **T1 · Illegal states unrepresentable.** Discriminated unions with one literal
  tag (`kind` for variants, `status`/`state` for a lifecycle, `ok` for an outcome),
  never a boolean beside an optional field. Exhaustive `switch` ending in
  `unreachable(x: never)`, or a `Record<Union, X>` table. *Caught by:* types.
- **T2 · No `any`, no `!`, no `as` on outside data, no `as unknown as`, no `enum`.**
  `unknown` at the edge, narrowed by a schema; handle the `undefined` that
  `noUncheckedIndexedAccess` reports; `satisfies` to check a literal without
  widening; `as const` object plus derived union instead of `enum`. The one
  sanctioned cast is inside a parse function that mints a branded type. *Caught
  by:* lint (`any`, `!`, `enum`), types (`erasableSyntaxOnly`), review (casts).
- **T3 · Derive, never re-declare.** `z.infer`, ORM row types, `Pick`/`Omit`,
  `ReturnType`. A second declaration of a shape drifts. `type` over `interface`
  unless declaration merging is the point. *Caught by:* review.
- **T4 · No hardcoded meaningful strings.** A status, kind, reason, capability,
  config key, job name, event name, route or storage key is declared once as an
  `as const` object with its union and referenced everywhere. *Caught by:* review.
- **T5 · Data is readonly.** `readonly` fields and `readonly T[]` parameters;
  return new values; no module-level mutable state. *Caught by:* review.

## Boundaries, errors and edges — `references/types-errors-edges.md`

- **E1 · Parse at the edge, trust inside.** Request bodies, query strings, env,
  vendor responses, webhooks, files, LLM output and stored JSON are parsed once by
  a schema (zod). Inside, no re-validation and no casts. *Caught by:* review.
- **E2 · Expected failures are returned, bugs and outages thrown.** A use case
  returns its own union (`{ ok: true, … } | { ok: false, reason: 'literal' | … }`),
  plain data, no `Result` class. Throws are caught once, at the transport (route
  handler, server action, job runner, CLI `main`) or in an adapter translating a
  vendor error. A `catch` that swallows, returns `[]` or logs-and-continues in a
  write path is a finding. *Caught by:* test, review.
- **E3 · Errors leave in one shape.** HTTP failures as `application/problem+json`
  with a `detail` a person can act on; an unexpected error is logged with its
  request id and answered with a fixed 500, never the message or the stack.
  *Caught by:* test, review.
- **E4 · Dependencies point inward; dependencies are parameters.** Pure core knows
  no framework, database, HTTP, clock or vendor. A use case takes `(input, deps)`;
  one wiring file builds real adapters; no DI container. A package exposes one
  public entry (`exports` map); nothing imports another package's internals.
  *Caught by:* lint (`noRestrictedImports`), types (`exports`), review.
- **E5 · One vendor, one adapter.** Vendor words and wire shapes stay in its
  adapter; every call has a timeout; a vendor failure degrades, never blocks the
  flow. *Caught by:* review.
- **E6 · Logs are structured and clean.** One logger with a request id; no
  `console.*` outside CLI scripts; never a password, token, key, session id,
  cookie or request body. *Caught by:* review.

## Env and config — `references/env-and-config.md`

- **C1 · Env is for secrets and process wiring.** Validated by a zod schema per
  group, every problem listed at once, values never in the error. Each app asserts
  the groups it needs at boot, so a bad deploy dies on start, not mid-request.
  *Caught by:* test, review.
- **C2 · `.env.example` lists every key**, with no real values; a test compares it
  to the schemas. `.env` is git-ignored. *Caught by:* test.
- **C3 · Anything that varies per tenant or gets toggled over time** lives in
  runtime config (database), not in code constants or env. *Caught by:* review.
- **C4 · Money and dates have one path.** Never a JS float for money: integer minor
  units or decimal strings, one representation per project, through one helper
  module. "Today" is the business's day in its time zone, never
  `toISOString().slice(0, 10)`. *Caught by:* test (Bun runs tests in UTC), review.

## Tests — `references/testing.md`

- **Q1 · A test is a behaviour sentence that can fail.** Revert the change in your
  head: if no test fails, the test is decoration. A bug fix starts with a failing
  test. No `.only`/`.skip` committed. *Caught by:* review, lint.
- **Q2 · Pure core gets unit tests; adapters get round-trip tests against the real
  thing** (a throwaway Postgres, not a mocked `query`). Never mock our own code.
  *Caught by:* review.
- **Q3 · A suite whose dependency is missing fails, it does not skip.** A green run
  that did no work is worse than a red one. *Caught by:* hook.

## After you write

1. `bunx biome check --write <files>`, then the repo's gates on the files you wrote.
2. Run what proves the change: typecheck, the tests it touches, the build where
   one exists (the build catches what typecheck and tests cannot).
3. Read your own diff once as a reviewer: a comment added, a cast, an export nobody
   calls, a hardcoded string, a swallowed error. Everything marked *review* above
   depends on this read.
4. Commit per `references/git.md`: one line, only your own files.

## Adopting this in a repo

New repo: copy `assets/` (tsconfig base, `biome.json`, the two hooks), set
`"prepare": "git config core.hooksPath .githooks || true"`, add `knip`. Existing
repo: `references/adoption.md` — census before you configure, ratchet instead of
sweeping, and never land a rule without its check.

## Changing the rules

When a review finds the same class of mistake twice, build the check instead of
repeating the finding: a type or schema first, then a test that runs the thing,
then a source scan. Prove the check fires on a planted violation, and make a run
that measured nothing fail. Change a rule and its check in the same commit.
