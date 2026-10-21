# Concurrency and ownership

## Isolation before annotations

- Determine which state belongs to the UI, a worker or an external framework.
  Mark the UI owner `@MainActor`; keep shared mutable worker state in an actor
  when needed. Use immutable, Sendable values to communicate between them.
- Do not send framework-owned reference objects between actors simply because
  their containing DTO is marked `@unchecked Sendable`. Extract supported value
  snapshots at the owning boundary instead.
- Confirm target default actor isolation. An implicit default can change where
  apparently ordinary declarations execute; do not assume `nonisolated` or
  `async` means background execution across compiler configurations.
- `Task {}` can inherit actor isolation. Move expensive synchronous parsing or
  blocking I/O to a deliberately designed execution boundary supported by the
  project's toolchain. Do not scatter `Task.detached` as a responsiveness fix.
- Never block a cooperative executor with semaphores while waiting for future
  asynchronous work. Use structured asynchronous APIs or an appropriate bridge
  for genuinely blocking legacy I/O.

## Task lifetime

- Use `.task` for work whose lifetime matches a SwiftUI view. Use an explicit
  service owner for work that must outlive transient popovers or windows.
- Give long-lived tasks a cancellation path when the account, service or app
  session ends. Avoid a task retaining its owner indefinitely; a deinitializer
  cannot break a cycle that prevents it from running.
- Cancellation is cooperative. Check it around expensive work, propagate it,
  and prevent an older request from replacing newer state after an `await`.
  Actor isolation does not prevent this logical reentrancy problem.
- Decide how an error terminates, retries or degrades a stream. Do not restart a
  failed task forever without a bound or report success after swallowing errors.
- Bridge callback APIs with continuations only when exactly-once resumption and
  cancellation/lifetime behavior are accounted for. Prefer an existing async API.

## Interop exceptions

`@preconcurrency import`, `@unchecked Sendable` and `nonisolated(unsafe)` are
assertions the compiler cannot fully check. Use them only for a demonstrated
interop requirement with a small boundary, documented synchronization invariant
and an appropriate test or runtime exercise. They are not a general diagnostic
suppression strategy. Follow stricter project rules when present.

## Evidence

Compiler success proves the checked isolation rules, not every lifecycle or
logical race. Test cancellation, stale results and repeated start/stop behavior.
Exercise UI responsiveness with real work. Use available runtime diagnostics
for demonstrated concerns, and state their coverage limits.

- [Swift concurrency](https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html)
- [Swift 6 data race safety](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/dataracesafety/)
