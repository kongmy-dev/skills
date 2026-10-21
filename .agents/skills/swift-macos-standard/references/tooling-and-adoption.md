# Tooling and adoption

## Confirm the actual toolchain

Run `xcodebuild -version`, `xcrun swift --version` and
`xcrun swift-format lint --help`. A compiler release and a Swift language mode
are different settings. Record a supported Xcode release in the project and
select it in CI; do not assume every developer's active toolchain matches.

For new first-party Xcode targets, merge
[StrictSwift.xcconfig](../assets/StrictSwift.xcconfig) into their configuration
and connect it to every relevant Debug/Release configuration. Merely committing
an xcconfig does not activate it. Confirm effective `SWIFT_VERSION` and
`SWIFT_TREAT_WARNINGS_AS_ERRORS` using `xcodebuild -showBuildSettings` for the
actual project/workspace and scheme. Swift 6 mode already enables strict
concurrency checking; no redundant upcoming-feature flag is needed.

For SwiftPM, select Swift 6 language mode in the manifest using the package's
supported tools API. Use `swift build -Xswiftc -warnings-as-errors` and
`swift test -Xswiftc -warnings-as-errors` for an appropriate standalone package.
These command-line flags can affect dependencies too; when that is disruptive,
scope enforcement to owned targets using supported settings rather than editing
vendor code or claiming those flags are first-party-only.

## Formatting

For a new project, copy [the formatter configuration](../assets/.swift-format)
to its root `.swift-format`. It uses toolchain defaults plus explicit style and
force-unwrap/force-try checks. Check supported rule names with
`xcrun swift-format dump-configuration` on the selected toolchain.

For explicit files:

```sh
xcrun swift-format format --configuration .swift-format --in-place Sources/Example.swift
xcrun swift-format lint --configuration .swift-format --strict Sources/Example.swift
```

Use actual owned file paths or source/test directories. `--recursive` is useful
for a known source root; avoid scanning dependencies, generated sources, shared
skills or build output. No path means stdin, not "the entire project". Confirm
that a selected directory actually contains Swift before reporting success.
Formatting does not enforce every semantic rule and does not fix all lint rules.

## App verification

Discover a real scheme using `xcodebuild -list` for the existing project or
workspace. Build and test that scheme with a macOS destination appropriate to
the target. Keep build artifacts in a designated output directory; never clear
shared DerivedData or package caches to work around a failure.

An unsigned build may validate compilation, but does not validate the intended
signed app's permissions, entitlements, background behavior or distribution.
Do not invent team IDs, disable signing tests or install hooks that silently
skip required tools. Preserve existing Git hooks; wire checks into the project's
existing verification commands/CI when those targets exist.

## Existing repositories

1. Inspect current target settings, minimum macOS, formatter and tests.
2. Adopt formatting without sweeping unrelated files.
3. Enable compiler checks per target and resolve the findings in bounded changes.
4. Keep any required interop exception at its boundary with a reason and an
   owner. Do not grant a permanent repo-wide escape from concurrency checking.
5. Add semantic checks only when they catch demonstrated defects. Report what
   remains review-only or unproven.

## Sources

- [Swift 6 migration guide](https://www.swift.org/migration/)
- [swift-format configuration](https://github.com/swiftlang/swift-format/blob/main/Documentation/Configuration.md)
- [Swift API naming guidance](https://www.swift.org/documentation/api-design-guidelines/)
