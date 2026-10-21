---
name: swift-macos-standard
description: >-
  Coding standard for native macOS apps and their Swift packages: Swift 6 types,
  concurrency and actor isolation, SwiftUI/AppKit ownership, permissions,
  boundaries, errors, formatting, tests and adoption. Use before writing or
  reviewing Swift in a Mac app, scaffolding a native macOS target, changing
  Package.swift or Xcode build settings, or answering requests for Swift coding
  standards, strict concurrency, native app conventions or macOS lifecycle fixes.
  Not a standard for server-side Swift, iOS-only apps or TypeScript projects.
license: MIT
metadata:
  author: kongmy
  version: '1.0'
---

# Swift/macOS standard

Write idiomatic Swift for native Mac apps. The user's instructions and the
project's own rules take precedence; disclose material conflicts and resolve
them from that authority. Do not force the neighbouring code's old patterns into
new work or expand an authorized change into an unrelated migration.

Each rule names its actual check: **compiler**, **formatter**, **test**,
**runtime** or **review**. A review-only rule is not an automated gate.

## Before writing

1. Identify the caller, input, state transitions and observable result. State
   material assumptions briefly and choose the smallest complete change.
2. Reuse existing types and behavior. Give new abstractions an immediate caller;
   use protocols for a real polymorphic or test boundary, not for every type.
3. Confirm the selected Xcode/Swift toolchain, macOS deployment target and build
   system. Resolve unfamiliar or evolving APIs with `ctx7 library` followed by
   `ctx7 docs`; if unavailable, use official Apple or Swift documentation.

## Toolchain and enforcement

Read [tooling and adoption](references/tooling-and-adoption.md) when setting up
compiler options, formatting, build commands or an existing-project migration.

- **B1 · Native tools first.** Use Xcode/xcodebuild for app targets, SwiftPM for
  packages, and system frameworks where they fit. Record the Xcode release and
  deployment target the project supports. Bun applies only to auxiliary
  JavaScript tooling; it does not replace SwiftPM. *Check: build, review.*
- **B2 · Swift 6 language mode for new targets.** A Swift 6 compiler alone does
  not enable this mode. Treat first-party Swift warnings as errors; migrate
  existing targets incrementally instead of disabling isolation diagnostics.
  *Check: compiler, effective build settings.*
- **B3 · One formatter.** Use the toolchain's `swift-format` and a checked-in
  `.swift-format`. Lint with `--strict`; do not run competing formatters. Respect
  an existing formatter until a migration is authorized. *Check: formatter.*

## Types, names and shape

- **T1 · Model valid states.** Use structs and `let` by default; use enums with
  associated values for mutually exclusive states. Exhaustively switch over
  owned enums; handle `@unknown default` for non-frozen external enums. Avoid
  boolean/optional combinations that permit impossible states. *Check: compiler,
  test, review.*
- **T2 · No forced recovery.** Unwrap with `guard`/`if let` or model absence.
  Do not add force unwraps, `try!`, force casts, silent `try?` or `fatalError`
  to recover from external input or ordinary failures. Required framework
  signatures and tool directives need narrow, justified treatment rather than
  blanket suppressions. *Check: formatter for force unwrap/try; review for casts
  and error handling; tests for failure paths.*
- **T3 · Swift names at the call site.** Use UpperCamelCase types and matching
  primary-type filenames, lowerCamelCase members, clear argument labels and
  meaningful units. Use `Date` for instants and `Duration` where supported for
  elapsed durations. Respect framework spellings and wire formats at adapters.
  *Check: formatter, review.*
- **T4 · Small, coherent ownership.** Keep access private/internal by default;
  introduce public API only for consumers outside the module. Split by behavior
  and ownership, not arbitrary line counts. Define reusable domain cases and
  storage keys once, without constant-wrapping every display literal.
  *Check: compiler access control, review.*
- **T5 · Code carries the explanation.** Prefer names, types and behavior tests
  to narration, section banners, commented-out code and historical notes. Keep
  necessary license/tool directives, concise platform constraints with a source,
  and API documentation when callers need a contract names cannot express.
  Preserve knowledge when changing older comments; do not bulk-delete them.
  *Check: review.*

## Boundaries and errors

- **E1 · Decode and validate at the edge.** Decode outside data into DTOs, then
  validate semantic constraints before constructing domain values. `Codable`
  checks representation, not business validity. Keep provider and framework
  objects in their adapters. *Check: compiler, boundary tests, review.*
- **E2 · Explicit dependencies.** Pass the needed client, clock or storage
  operation into logic. Keep decision logic independent of SwiftUI/AppKit,
  network requests and permission prompts. Prefer simple values/closures or a
  small protocol over a container or a singleton. *Check: tests, review.*
- **E3 · Preserve failure meaning.** Represent actionable states such as denied
  permission, expired credentials, unavailable service and empty results
  distinctly. Use `throws` and typed domain errors naturally; `Result` is useful
  when a result must be stored or passed. Catch at an adapter or presentation
  boundary, preserve cancellation, and never disguise failure as `[]`.
  *Check: tests, runtime, review.*
- **E4 · Bounded, private I/O.** Give requests timeouts and cancellation; retry
  only appropriate transient failures with a bound and backoff. Keep secrets in
  Keychain, ordinary preferences in an appropriate local store, and sensitive
  content out of logs and fixtures. Use structured `Logger` categories with
  explicit privacy decisions. *Check: failure tests, runtime, review.*

## Concurrency and native lifecycle

Read [concurrency](references/concurrency.md) for asynchronous work, shared state
or actor-isolation diagnostics. Read [native lifecycle](references/native-lifecycle.md)
for SwiftUI/AppKit state, menu bar utilities, timers, permissions or app adapters.

- **C1 · Isolation is a design boundary.** Keep UI state on `@MainActor` and
  shared mutable worker state actor-isolated. Pass `Sendable` snapshots across
  isolation boundaries. Do not use `@unchecked Sendable`, `nonisolated(unsafe)`
  or blanket `@preconcurrency` to make a diagnostic disappear.
  *Check: Swift 6 compiler; review for escape hatches.*
- **C2 · Every long-lived operation has an owner.** Prefer structured tasks;
  explicitly own and cancel long-lived tasks, streams, timers and observations.
  Guard against stale completions after cancellation or account changes.
  `async` alone does not move work off the main actor. *Check: lifecycle tests,
  runtime responsiveness, review.*
- **M1 · One source of presentation state.** Views describe state and user
  actions; services own fetching and side effects. Use Observation or
  ObservableObject according to the deployment target and existing design.
  AppKit delegates/coordinators bridge real native behavior without duplicating
  service state. Preserve keyboard access and accessible names for custom UI.
  *Check: UI tests, runtime, review.*
- **M2 · Permissions and wake are normal states.** Prompt at the relevant user
  action, provide denial/revocation recovery, and configure required usage
  descriptions/entitlements. Reconcile external state after wake or foreground
  changes. A scheduled tick is not proof a block of time elapsed or data synced.
  *Check: target configuration review and real runtime exercises.*

## Verification and completion

1. Test observable behavior at the narrowest useful level. Prefer Swift Testing
   for supported new unit targets; use XCTest where the target or UI automation
   requires it. Follow existing conventions. Add a regression test for meaningful
   bug fixes when practical, not tests that merely mirror simple implementation.
2. Inject time and use synthetic fixtures. Exercise boundaries, cancellation and
   state transitions. Unit fakes are useful; they do not prove provider access,
   TCC permissions, code signing, Keychain prompts or sleep/wake behavior.
3. Format only intended first-party files, lint, then build and run relevant tests
   for the real target. A package build does not validate the app bundle. Missing
   required tools, an empty test run or a placeholder target is **unproven**,
   never a pass. Do not bypass required checks to get a green result.
4. Review the changed files for isolation escapes, duplicated state, silent
   failures, sensitive logging and unnecessary abstractions. Keep unrelated
   changes out of the work. Stage named owned paths and commit under the project's
   Git rules; this skill does not authorize publishing, deployment or messages.

## Improving the standard

For repeated mistakes, prefer a compiler constraint, a type or a behavior test
before adding a source scan. Add a check only when it detects the real defect;
exercise it with a failing case. Do not claim a formatter checks architecture,
resource lifetime or data race behavior at unsafely annotated boundaries.
