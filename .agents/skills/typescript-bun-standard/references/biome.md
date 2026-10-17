# Biome

Biome is the formatter, the linter and the import organiser, in one pass.
`assets/biome.json` is the baseline for a new repo.

## Formatter

| Setting | Value | Why |
|---|---|---|
| indent | 2 spaces | |
| line width | 100 | Between the 80 that wraps every signature and the 120 that hides a second statement off-screen. |
| quotes | single (JSX double) | |
| semicolons | always | No ASI surprises on lines starting `(` or `[`. |
| trailing commas | none | |
| arrow parens | always | |

An existing repo doesn't take these blindly: count what the tree already does
(quote style, files with and without semicolons, line-length p95) and adopt the
majority. The smallest diff that buys consistency beats a preference.

## Linter

`"preset": "recommended"` plus these as errors:

| Rule | Backs |
|---|---|
| `suspicious/noExplicitAny` | T2 |
| `style/noNonNullAssertion` | T2: handle the `undefined` |
| `style/noEnum` | T2 / T4: `as const` object plus union |
| `correctness/noUnusedVariables`, `noUnusedImports`, `noUnusedFunctionParameters` | S3 |
| `suspicious/noImplicitAnyLet` | T2 |
| `suspicious/noAssignInExpressions`, `style/noParameterAssign` | readability |
| `style/noUselessElse` | guard clauses (S2) |
| `suspicious/noConsole` | E6 (off for scripts and tooling) |
| `suspicious/noFocusedTests`, `suspicious/noSkippedTests` | Q1 |

Test files relax `noExplicitAny` and `noNonNullAssertion`: a test walking parsed
JSON fails loudly when an assumption is wrong, which is a test's job. Shipped code
relaxes nothing.

Run it with `--error-on-warnings`, so a warning is never something to ignore.

## Boundaries with `noRestrictedImports`

A layer rule nobody can see is a rule you don't have. Encode each one as an
override on the paths it governs, with a `message` that says what to do instead:

```json
{
  "includes": ["**/src/core/**/*.ts"],
  "linter": {
    "rules": {
      "style": {
        "noRestrictedImports": {
          "level": "error",
          "options": {
            "patterns": [
              {
                "group": ["pg", "postgres", "react", "next", "next/**", "../adapters/**", "../service/**"],
                "message": "core/ is pure: no database, framework or sibling layer. Move this to service/ or adapters/."
              }
            ]
          }
        }
      }
    }
  }
}
```

Typical rules worth encoding:

- Pure core imports no framework, database, HTTP client, queue or sibling layer.
- Apps import a package's public entry only, never `@scope/pkg/internal/...`.
- Only the database package imports the database driver.
- The design system imports no domain package.

When the rules need a graph (cycles, orphans, "nothing outside X imports Y"), add
`dependency-cruiser` and run it as a gate. Don't keep both a Biome rule and a
depcruise rule for the same boundary.

## Includes and generated code

- `vcs.useIgnoreFile: true` so `.gitignore` is respected.
- Exclude generated code (`!**/*.gen.ts`, ORM migrations, generated clients) and
  vendored code. Formatting a machine's output makes the generator's next run a
  diff.
- A negation never ends in a slash: `!packages/vendored` and
  `!packages/vendored/**` both exclude, `!packages/vendored/` parses and matches
  nothing.

## Suppressions

No `biome-ignore` in shipped code. If a rule is wrong for a whole area, change the
config with an override, in a commit that says why. If it's wrong for one line,
the line is usually what's wrong.
