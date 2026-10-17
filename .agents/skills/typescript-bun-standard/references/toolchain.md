# Toolchain

## Bun does everything the package manager and test runner used to

| Job | Command | Not |
|---|---|---|
| install | `bun install` (CI: `bun install --frozen-lockfile`) | `npm ci`, `pnpm i` |
| add a dependency | `bun add <pkg>`, `bun add -d <pkg>` | `npm i` |
| global CLI | `bun add -g <pkg>` | `npm i -g` |
| one-off binary | `bunx <pkg>` | `npx` |
| script | `bun run <script>`, `bun <file>.ts` | `ts-node`, `tsx` |
| one workspace | `bun --filter <pkg> <script>` | `cd packages/x && …` |
| tests | `bun test` | Jest; Vitest only where a framework needs it, with the reason in `CLAUDE.md` |

Why one tool: one lockfile, one resolver, one cache. Mixing managers produces two
lockfiles that disagree, and `npx` resolves a different copy of a CLI than the one
the lockfile pins.

## Root `package.json`

```json
{
  "private": true,
  "packageManager": "bun@1.4.2",
  "workspaces": ["apps/*", "packages/*", "tooling/*"],
  "scripts": {
    "typecheck": "tsc -b",
    "lint": "biome check --error-on-warnings .",
    "lint:fix": "biome check --write .",
    "test": "bun test",
    "deadcode": "knip --no-progress",
    "prepare": "git config core.hooksPath .githooks || true"
  },
  "devDependencies": {
    "@biomejs/biome": "2.5.14",
    "@types/bun": "1.4.2",
    "knip": "6.37.0",
    "typescript": "6.0.3"
  }
}
```

- `packageManager` pins Bun so every machine and CI agree on lockfile format.
- Exact versions for the tools that own config: a Biome minor adds rules and moves
  its schema, a TypeScript minor changes diagnostics. Renovate or a deliberate bump
  moves them, never a caret.
- Never `@types/bun: "latest"`: it changes under you on the next install.
- In a monorepo with more than a handful of packages, Turbo fans the scripts out
  (`turbo run typecheck lint test --continue`). Treat the fan-out as correctness —
  every package's gate must run — not as a cache.

## `bunfig.toml`

Only what the repo actually needs. Common entries:

```toml
[install]
exact = true

[test]
preload = ["./test-setup.ts"]
```

- `exact = true` makes `bun add` write exact versions (B2).
- `bun test` runs in UTC (`Etc/UTC`) by default. Leave it: code that reads the
  machine's zone then fails everywhere, which is the point.
- Bun loads `.env` on its own. Don't let correctness depend on that: a script that
  needs secrets names its file (`bun --env-file=.env scripts/x.ts`), so the
  dependency is visible, and tests never read the real `.env`. `env = false` in
  `bunfig.toml` turns the autoload off where that matters.
- Dependency lifecycle scripts don't run unless the package is in
  `trustedDependencies`. Add a package there only when its install script is the
  reason you depend on it.

## Runtime

Bun is the toolchain. Whether production runs on Bun or Node is a decision per
deployable, written in the repo's `CLAUDE.md`. Next.js, for example, builds and
serves on Node even when Bun installs and runs everything else. Don't use a
`Bun.*` API in code that ships to a Node runtime.

## Git hooks

`.githooks/` committed, wired by the `prepare` script (no Husky dependency).

- **pre-commit** (`assets/githooks/pre-commit`): Biome on the staged files only.
  Fast enough that nobody disables it.
- **pre-push** (`assets/githooks/pre-push`): the gates CI runs that are fast and
  offline: typecheck, lint, test, build, deadcode. It runs every gate and reports
  all failures, because stopping at the first red hides the second. It skips a
  push that only deletes remote refs, and it warns when the tree is dirty, since a
  failure may belong to uncommitted work that isn't being pushed.
- Skipping with `--no-verify` is for emergencies. If it becomes routine, the hook
  is wrong: fix the hook.

## CI

Same gates as pre-push, plus anything slow or online (dependency audit, coverage
ratchet, static analysis). Install with `oven-sh/setup-bun` and
`bun install --frozen-lockfile`, so a `package.json` that drifted from `bun.lock`
fails instead of silently resolving.
