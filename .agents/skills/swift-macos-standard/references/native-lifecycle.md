# Native macOS lifecycle

## SwiftUI and AppKit

- Put durable application services at app/session scope. A menu bar popover may
  be recreated whenever it opens; it must not silently duplicate synchronization
  or stop essential app work whenever it closes.
- Keep a single observable owner for presentation state. With a supported
  deployment target, Observation can simplify updates; do not mechanically
  replace working ObservableObject ownership or add another store beside it.
- Keep side effects out of `body`. Preserve stable identity for list rows and
  view-owned state. Separate AppKit delegate/coordinator bridging from domain
  decisions, and honor required delegate ownership and weak-reference rules.
- Verify keyboard shortcuts, focus, accessible labels and dismissal behavior in
  the running app. A screenshot or a SwiftUI preview cannot prove those work.

## Time and external change

- For a deadline tied to calendar time, derive remaining time from the current
  instant and deadline. For elapsed intervals, use a suitable monotonic clock.
  Do not decrement a saved counter and assume every tick was delivered.
- Observe wake, relevant calendar/store changes and account changes when the
  feature needs them. Re-fetch or reconcile state rather than replaying missed
  ticks. Refresh stale framework objects according to their API contracts.
- Inject time into decision logic. Cover exact boundaries, clock adjustments,
  repeated events and timezone/DST cases relevant to the feature. Keep date-only
  concepts distinct from instants; a fixed 24-hour duration is not every local day.
- The app cannot guarantee continuous work during system sleep. Distinguish a
  local scheduled notification from an app process remaining awake.

## Permissions, storage and adapters

- Request the smallest supported access at the feature's point of use. Some
  APIs have broader permissions than the behavior needs; explain the actual
  permission instead of promising a narrower scope the platform lacks.
- Model not-determined, denied/restricted, authorized and later-revoked states as
  relevant to the API. Offer a useful recovery path without repeated prompting.
- Check usage descriptions, entitlements, sandbox rules, signing identity and
  availability against the exact framework and target. Keep the intended app
  identity stable during permission testing; a command-line helper's grant does
  not prove the signed app has access.
- Validate Keychain behavior in the intended app identity. Store preferences
  separately from secrets. Do not put sensitive content in notifications,
  clipboard history, logs or telemetry by accident; choose exposure deliberately.
- Use supported APIs for other apps. Treat scripting dictionaries as candidate
  interfaces, then verify actual commands with consent and minimal data exposure.
  Do not build background sync on undocumented databases or fragile UI scraping.
- Register observers and callbacks once per owner, and remove/invalidate them
  at the corresponding lifecycle boundary. Handle reconnects without duplicates.

## Verification boundary

Use synthetic values for unit and UI tests. Real permission prompts, native
integration, signing, reconnects, lock/wake and provider behavior need a separate
runtime exercise when they matter. Report **pass**, **fail** and **unproven**
individually; a green parser test does not prove an inbox connection.

- [Apple app automation permissions](https://support.apple.com/guide/mac-help/mchl108e1718/mac)
- [Apple EventKit access](https://developer.apple.com/documentation/eventkit/accessing-the-event-store)
- [Apple Task documentation](https://developer.apple.com/documentation/swift/task)
