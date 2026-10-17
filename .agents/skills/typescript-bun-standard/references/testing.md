# Testing

Tests are the ship gate. A test earns its place by being able to fail for the
reason its name gives.

## What gets which test

| Code | Test | Runner |
|---|---|---|
| pure core: rules, calculations, state machines | unit tests, many, fast | `bun test` |
| a use case | plain fakes passed as `deps` | `bun test` |
| an adapter that talks to a database | round-trip against a real throwaway Postgres, one rolled-back transaction per test | `bun test` |
| an adapter that talks to a vendor | recorded responses parsed by the real wire schema, plus a contract check against the vendor's docs | `bun test` |
| an HTTP route | status, problem body and stored effect | `bun test` |
| a user flow | end-to-end in a browser | Playwright |

Mocking `pool.query` proves the mock was called, not that the SQL is right. Never
mock our own code: pass a fake through `deps` instead, which is what parameters are
for.

## A test is a sentence that can fail (Q1)

- Name it for the behaviour: `"a receipt over the tolerance waits for a manager"`,
  not `"calls save"`.
- Revert the change in your head. If no test fails, the test is decoration.
- A bug fix starts with a test that fails for the bug.
- Assert on outcomes, not on calls.
- No `.only`, no `.skip` committed (Biome: `noFocusedTests`, `noSkippedTests`).
  No loop over an empty list and no filter that matches no file: a test that
  asserted nothing passes.

## Isolation

- Every test builds its own fixtures. Never depend on another test's data or on
  run order.
- Multi-tenant code: every repository suite has a test with two tenants, proving
  one can't read or write the other's rows.
- Time: inject `now`, or use `setSystemTime` from `bun:test`. Tests run in UTC.
- Randomness and ids: inject them where a test asserts on them.

## Missing dependencies fail (Q3)

A database suite with no database fails and says how to start one. It doesn't
skip. Skipping silently turns the gate green for work it never did. If someone
genuinely can't run it, an explicit opt-out (`ALLOW_DB_SKIP=1`) makes the skip a
choice they made, not a default nobody noticed.

## Running

- `bun test <path>` for the files you touched while working; the full suite before
  push (the pre-push hook runs it).
- In a monorepo, run every package's suite through the fan-out (`turbo run test
  --continue`), so one package's failure doesn't hide another's.
- Coverage is a ratchet owned by CI (it may go up, never down), not a number to
  chase. `bunfig.toml` `coverageThreshold` works for a single package.
