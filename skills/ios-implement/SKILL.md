# iOS: Implement code changes for a ticket

$ARGUMENTS

This skill handles the **iOS-specific** implementation steps. Shared steps (requirement confirmation, review gate) are in `/mobile-clarify`.
For design system component mapping, use `/ios-cds`.

## Prerequisites

Run `/mobile-clarify` first to complete:
- Requirement understanding and confirmation (Step 1 + 1b)
- Figma design review (if UI ticket)

Only proceed here after the user has approved the requirement confirmation.

## KMP-Aware Implementation

When KMP changes were made (via `/kmp-implement`), this skill can start **immediately** without waiting for the KMP snapshot to finish publishing. Use your knowledge of the KMP changes (new types, APIs, models, use cases) to write the iOS code.

- **During implementation (Steps 1-4)**: Write code based on the KMP types/APIs you know were created. You have context from the KMP implementation about what was added/changed.
- **During local verify (Step 5)**: The KMP snapshot version is needed to run `pod install` and build. If the snapshot is not yet available, **ask the user for the KMP snapshot version** before proceeding with Step 5.

## Step 1: Research

Before writing any code:
1. Search for existing protocols/classes/models mentioned in the ticket: `Glob **/<Name>.swift`
2. Check `PodLocals/` for external pod protocols — they already exist, NEVER recreate them
3. Read ALL files that will need changes
4. Identify the architecture pattern used (coordinator, MVVM, DI assembly, etc.)
5. Check if the repo uses Cuckoo mock generation script (`script/iosCuckooGenerateMocks*.sh`)
6. **If the ticket involves KMM shared version changes**: run `pod install` first, then verify new KMM types exist by grepping the shared framework headers:
   ```bash
   grep "TypeName" PodLocals/*/Current/shared.xcframework/ios-arm64-simulator/shared.framework/Headers/shared.h
   ```
   This confirms what types/methods are available before writing code against them.

## Step 1b: Design system mapping (if UI ticket) → `/ios-cds`

If the ticket involves UI changes, run `/ios-cds` to map Figma components to their implementations in `tc-mx-ios-design-component`. This ensures correct component usage before writing any UI code. Do NOT improvise DS components — if a component isn't found, `/ios-cds` will ask for clarification.

## Step 2: Plan

- List each file to modify and what changes
- For UI changes: reference the `/ios-cds` mapping to confirm which DS components to use
- Confirm scope: which flows to touch, which to leave alone
- If adding new dependencies: trace the DI path (protocol → assembly → resolver)
- **If changing a protocol in an Api pod**: check what downstream repos depend on. They only see the Api pod — any new parameter/return types must be available in the Api pod. Don't expose impl-pod concrete types (e.g., custom UIView subclasses) through the protocol. Use callbacks/closures instead.
- **If touching multiple repos**: run `pod install` in ALL affected repos (sub-module, downstream, main app) before building — stale Pods.xcodeproj references cause phantom "file not found" errors
- Present the plan and ask for confirmation

## Step 3: Implement

- Follow existing patterns in the codebase
- Don't modify flows/methods not related to the ticket
- Don't add fields to existing models unless explicitly required
- Refactor large methods into focused private methods
- If new files are needed, check that they don't already exist first
- **KMM sealed class mapping**: ALWAYS use `switch` with `case let x as SubClass` / `case is SubClass` / `default:` — NEVER use if-else chains for KMM type checks
- **Podspec version bumping** (MANDATORY): Only bump podspecs for modules **impacted by the code change** — not all modules in the repo. Bump the **patch number** per ticket (e.g., 2.0.0 → 2.1.0, 2.3.0 → 2.4.0). Do this before committing. This applies to EVERY ticket — even small changes need a version bump for the snapshot to publish a new version.
- **CLAUDE.md**: Many repos gitignore `.claude/` — check `.gitignore` before trying to commit `.claude/CLAUDE.md`. If gitignored, CLAUDE.md is local-only guidance for Claude Code.

## Step 4: Tests

- Use the repo's mocking framework (Cuckoo for most TymeX repos)
- If a new protocol needs mocking: update the mock generation script, regenerate, then write tests
- Add tests for each new code path
- Follow existing test naming convention in the repo
- For testing callbacks/closures: stub the caller to invoke the callback with test data
- For private `@objc` methods: test via `sut.perform(NSSelectorFromString("methodName"))`

## Step 5: Local verify (mandatory — do NOT skip or stop early)

### 5-pre. KMP snapshot integration (if KMP changes were involved)

If implementation was done based on KMP knowledge without the actual snapshot:

1. **Ask the user for the KMP snapshot version** if not yet provided:
   > "iOS code changes are complete. Please provide the KMP snapshot version so I can update the Podfile and verify the real integration."
2. Update the `shared` version in `Podfile` / `Podfile.swift` to the KMP snapshot version
3. Run `pod install`
4. Run `xcodebuild clean` to clear stale module cache
5. Verify KMM types exist in the shared framework headers:
   ```bash
   grep "TypeName" PodLocals/*/Current/shared.xcframework/ios-arm64-simulator/shared.framework/Headers/shared.h
   ```

Then proceed with the remaining verify steps below.

Run ALL of the following checks in order. Do not stop until every check is done, even if one fails (report all results at end).

### 5a. Detect the CI test scheme (same logic as commitCheck.sh)
```bash
xcworkspace=$(find . -maxdepth 1 -type d -name "*.xcworkspace" | head -1 | sed 's|^\./||')
moduleName=$(basename "$xcworkspace" .xcworkspace)
TEST_SCHEME="${moduleName}Tests"
```
**Use this `TEST_SCHEME` for all test runs. Do NOT guess or hardcode scheme names.**
Example: `Payment.xcworkspace` → `PaymentTests`, `AccountsSelection.xcworkspace` → `AccountsSelectionTests`

### 5b. Regenerate Cuckoo mocks
```bash
bash ./script/iosCuckooGenerateMocks.sh
```

### 5c. SwiftLint
```bash
./Pods/SwiftLint/swiftlint lint --strict 2>&1 | grep -v ".claude/"
```
If `./Pods/SwiftLint/swiftlint` not found, fall back to `swiftlint lint --strict`.

### 5d. Build and run unit tests (primary repo)
```bash
xcodebuild -workspace "$xcworkspace" -scheme "$TEST_SCHEME" \
  -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16' test
```

### 5e. If touching multiple repos (e.g., Api pod + downstream consumer):
For EACH affected downstream repo:
1. Update `Podfile.swift` to point changed pods to local path (`path: "../<source-repo>"`)
2. **If KMM shared version differs**: also update `shared` version in Podfile.swift to match the upstream repo's version (both repos must use the same shared that has the new KMM types)
3. Run `pod install`
4. Run `xcodebuild clean` to avoid stale module cache errors from shared version changes
5. Regenerate Cuckoo mocks: `bash ./script/iosCuckooGenerateMocks.sh`
6. Detect and run the CI test scheme (same logic as 5a)
7. **After verifying**, revert Podfile.swift changes (they're temporary for local testing)

### 5f. Report all results before proceeding

## Log output

After completing all steps, present the **Implementation Review Gate** (from `/mobile-clarify`): summary of all changes, wait for user approval.

```
=== [ios-implement] ===
Status: PASSED | FAILED
Requirement: <1-line summary>
Country target: <GTZA/SLZA/GTPH/unchanged>
Files changed: <count and list>
Files created: <count and list>
Tests added: <count and list>
```

## Next steps
- Run `/ios-precheck` to validate before PR (only after user approves implementation)
- Run `/bitbucket create pr` to commit and open PR
- Run `/mobile-snapshot` to publish snapshot version
- Run `/ios-verify` to verify on main app
