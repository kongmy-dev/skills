# Shape and comments

## No comments (S1)

Code carries no comments. A reader learns *what* from names, types and structure,
and *why* from a decision record, a test name or the commit. A comment is a defect
report: it marks a place where the code failed to say something, and the fix goes
in the code.

Why:

- **Comments drift and nothing catches it.** The compiler checks a type; nothing
  checks a sentence. A docstring promising a module "holds no secrets" can stay
  for months while the opposite becomes true and every gate passes.
- **Comments hide design problems.** A paragraph on when two fields may combine is
  a union that's missing. A paragraph on a surprising branch is a name or an
  extraction that's missing.
- **Every reader pays for them on every visit**, agents included.

### The fixed exceptions

| Exception | Form |
|---|---|
| a foreign constraint: behaviour forced by a vendor, a protocol, a browser or a runtime | `// vendor: <link> <one line>` |
| a decision a reasonable reader would "fix", that no type or test can express | the bare decision ID: `// ADR-012` |
| a licence or provenance header vendored code must carry | `/* SPDX-License-Identifier: … */` |

Fixed forms let a gate tell an exception from a comment. Directives such as
`'use client'` aren't comments. Generated files belong to their generator.

### Never

Narration of the next line; history ("used to", "removed", "who asked"); section
banners (a file that needs sections is several files); doc comments on our own
exports (rename instead); `TODO`/`FIXME` (open work goes to the backlog);
commented-out code (git keeps it); "IMPORTANT, do not remove".

### Where the knowledge goes instead

| It is… | It goes to |
|---|---|
| what a value is, or its unit | a name: `SESSION_IDLE_MS`, `amountMinor` |
| when a combination of fields is valid | a union |
| why a branch exists | a named predicate or an extracted function |
| an invariant | a test whose name states it |
| a decision and its reason | the repo's decision record (ADR or decisions log) |
| behaviour forced by a tool or vendor | a traps/gotchas doc, or a `// vendor:` line |
| work not done | the backlog |
| narration, history, who asked | nowhere: git has it |

### Existing comments: ratchet, never sweep

The comments already in a tree hold real knowledge: decisions, traps, reasons
written nowhere else. Bulk-stripping them loses it. When you rewrite code around a
comment, read it, move what it knows to the right row above, then delete it. A
gate that stops a file *gaining* comment lines (an edited comment counts as new)
holds the line while the old ones pay down.

## Write to the gate's shape (S2)

A size gate measures per function: cognitive complexity, length and nesting; per
file: code lines. Set its limits to the repo's own p95 when it's adopted. Never
copy limits from another repo, never raise them to bless a file, never exclude a
path to dodge them. Test files keep the complexity limit but drop length.

Shapes that fit:

- **Guard clauses.** Refuse the bad case first and return; the happy path stays
  flat. Four guards in one function means it has two jobs.
- **A table instead of branches.** `Record<Union, Value>` costs nothing and stops
  compiling when a variant is added.
- **A named predicate** for any condition with more than one operator:
  `const isOverdue = (invoice: Invoice) => …`. It reads as a word at the call site.
- **One job per function**, named for the job. A command is a sequence of named
  steps: `assert…` refuses, `apply…` writes, `record…` audits.
- **Array methods for transforms**; a loop only for side effects.
- **`?.` and `??`** instead of `if (x) { y = x.z }`.
- **No nested ternary.**
- **Split a file on its exports**: a name with its own callers and its own test is
  a file.

What not to do: don't wrap a statement in a one-caller helper just to lower its
score, and don't split by line count. A split that makes a reader open three files
to follow one thing made it worse.

## Subtract before you add (S3)

Judge a change by what the reader must trace. Delete first. Every new export has a
consumer in the same change; `knip` in CI fails on an unused file, dependency or
export. No:

- interface with one implementation
- factory that constructs one thing
- options object read in one place
- wrapper with one caller
- re-export shim or compatibility path kept "just in case"
- two ways of doing one thing side by side

## Names (S4)

| Thing | Rule | Example |
|---|---|---|
| file | kebab-case, named for its one job | `payment-due.ts` |
| function | verb phrase for an action, noun phrase for a derivation | `billOrder`, `paymentDue` |
| boolean | `is`/`has`/`can`/`should` | `canRetry` |
| number | unit as suffix | `timeoutMs`, `ttlSeconds`, `amountMinor` |
| time | `At` instant, `On` date | `capturedAt`, `dueOn` |
| collection | plural | `settlements` |
| type | noun, no `I` prefix, no `Type` suffix | `Settlement` |
| fixed constant | `SCREAMING_SNAKE` | `PAGE_LIMIT` |

Banned: `data`, `info`, `item` (outside a list callback), `utils`, `helpers`,
`misc`, `common`, `manager`, a bare `handle`. If the right name is long, the thing
does too much or isn't understood yet.

`snake_case` in SQL, `camelCase` in TypeScript; the ORM maps between them.
