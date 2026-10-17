# tsconfig

One base config (`assets/tsconfig.base.json`), extended by every package. In a
monorepo it lives in a `tooling/typescript-config` package or at the root as
`tsconfig.base.json`; the root `tsconfig.json` lists packages as `references`
and `tsc -b` checks them all.

## The flags and why

| Flag | Why |
|---|---|
| `strict` | Default in TypeScript 6; stated anyway so the config reads the same on 5.x. |
| `noUncheckedIndexedAccess` | `arr[0]` and `record[key]` are `T \| undefined`. The bug it prevents (a missing row read as present) is the most common one strict alone lets through. Pair it with handling the `undefined`, not with `!`. |
| `exactOptionalPropertyTypes` | `{ a?: string }` no longer accepts `{ a: undefined }`, so "absent" and "explicitly cleared" stay distinct. On by default in new repos; in an existing repo turn it on when the error count is small enough to fix in one change. |
| `noImplicitOverride` | A method that shadows a base method says `override`, so a renamed base method breaks the build instead of leaving a dead override. |
| `noFallthroughCasesInSwitch` | A missing `return`/`break` is an error. |
| `verbatimModuleSyntax` | Type-only imports must say `import type`. What you write is what gets emitted, which is what every transpiler (Bun, esbuild, Node type-stripping) assumes. |
| `erasableSyntaxOnly` | Bans `enum`, `namespace` with values and constructor parameter properties: syntax that can't be stripped. Code then runs under Node's type-stripping as well as Bun, and the no-`enum` rule has a compiler behind it. |
| `isolatedModules`, `moduleDetection: "force"` | Every file is a module, compiled on its own, as Bun compiles it. |
| `module: "Preserve"`, `moduleResolution: "bundler"` | Bun and bundlers resolve; `tsc` only checks. |
| `types: ["bun"]` | TypeScript 6 defaults `types` to `[]`. Without this, `Bun`, `bun:test` and `process` stop resolving. A package that ships to Node and not Bun uses `["node"]`. |
| `noEmit` | `tsc` is a checker. Bun runs the source; a bundler or `bun build` produces output. |
| `skipLibCheck` | Third-party `.d.ts` errors aren't ours to fix and slow every run. |

Unused locals and parameters are Biome's job (`noUnusedVariables`,
`noUnusedImports`, `noUnusedFunctionParameters`), not `tsc`'s: Biome can autofix
them and reports them in the same pass as everything else.

## Packages

```json
{
  "extends": "../../tooling/typescript-config/base.json",
  "include": ["src", "test"]
}
```

A React package adds `"jsx": "react-jsx"` and `"lib": ["ES2023", "DOM", "DOM.Iterable"]`.
A Next.js app extends the base and adds `"plugins": [{ "name": "next" }]`, `"jsx": "preserve"`.

## Things that aren't checked unless you check them

- Root scripts that belong to no workspace: give them a `tsconfig.json` and put it
  in the root `typecheck` script.
- Config files (`*.config.ts`): include them in some project.
- A package with no `typecheck` script: Turbo won't run what isn't declared.

## TypeScript 7

TypeScript 7 is the native (Go) compiler. It is much faster, but tools that use the
compiler's JavaScript API (custom gates, AST metrics) need TypeScript 6. A repo
that has such tools stays on 6 until they're ported; say so in `CLAUDE.md`.
Options deprecated in 6 (`target: ES5`, `outFile`, `module: UMD`) are removed in 7:
don't use them, and don't silence them with `ignoreDeprecations`.
