# Git

## Commits

- **One line**: `type: what changed`, under ~70 characters. Types: `feat`, `fix`,
  `refactor`, `test`, `docs`, `chore`, `perf`, `ci`, `build`.
- A body only for what the diff can't show: a non-obvious why, a trade-off, a
  rollback point. Then 1–3 short bullets. Never explain what the diff shows.
- No attribution lines.
- Commit each logical change when it's done and green. Don't batch a day's work.

## Staging in a shared checkout

Assume another session (a person or an agent) may be working in the same tree.

- Stage named files: `git add path/to/file`. Never `git add .`, `-A` or
  `commit -a`: they sweep up someone else's work in progress.
- Re-check `git status` right before staging; the tree moves.
- Never discard changes you didn't make: no blanket `git checkout .`,
  `git reset --hard`, `git clean -fd`, `git stash drop`. Prefer a stash
  (recoverable) when something blocks you.
- Don't clear shared caches (`node_modules`, `.turbo`, build output) unasked.
- Pull with `git pull --rebase --autostash`. On conflict, merge both sides; both
  are usually real work.

## Before push

The pre-push hook runs the gates. If it fails on a file you didn't touch and the
tree is dirty, check whether the failing path is in what you're pushing
(`git diff --stat @{push}..HEAD`). If it isn't, `--no-verify` is the right call;
otherwise fix it.

## Never in a commit

- Secrets, `.env`, cookies, tokens, real customer data.
- Generated output that the build produces (unless the repo commits it on purpose
  and says so).
- The name of another project, or where an idea came from. Carry the idea over and
  state it as this project's own.
