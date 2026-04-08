# iOS: Verify sub-module changes on main app

$ARGUMENTS

Verify that sub-module changes integrate correctly with the main app, fix cascading dependency issues, and generate manual test cases.

## Step 1: Identify the main app repo

Based on the country target from the ticket:

| Target | Main app repo |
|--------|--------------|
| GTZA | `tb-gosa-ios-app` |
| SLZA | `tb-sanlam-ios-app` |
| GTPH | `ph-goph-ios-app` |

If no country was specified, ask the user which main app to verify against.

## Step 2: Update main app to use local path

In the main app repo, update the `Podfile` (or `Podfile.swift`) to point to the changed sub-module's local path:
- Change `artifact: "ModuleName", version: "X.X.X"` → `path: "../<sub-module-repo>"`
- **If KMM shared version was updated in the sub-module**: also update `shared` version in the main app's Podfile to match. Mismatched shared versions cause "missing type" or "stale module" build errors.
- Run `pod install` to link locally
- Run `xcodebuild clean` before building if shared version changed (avoids stale precompiled module cache)

## Step 3: Build the main app

```bash
xcodebuild -workspace <App>.xcworkspace -scheme <AppScheme> -sdk iphonesimulator build
```

## Step 4: Handle build result

### If build SUCCEEDS → go to Step 5

### If build FAILS → analyze errors

1. **Failure is unrelated to sub-module changes** → Note the pre-existing failure and confirm sub-module changes are not the cause. Go to Step 5.

2. **Failure is caused by sub-module changes breaking another dependent repo** (e.g., sub-module API changed, and an intermediate module like `ios-payment` doesn't compile):
   - Identify which dependent repo is broken (check the error: which module/file fails)
   - Check if the source for that repo exists locally (e.g., `../tc-mx-ios-payment`)
   - If it exists locally:
     1. Jump into that repo
     2. Update its `Podfile`/`Podfile.swift` to point to the changed sub-module via local path
     3. Run `pod install`
     4. Implement the necessary adaptations to make it compile (follow same patterns as `/ios-implement`)
     5. Run `/ios-precheck` on that repo
     6. Commit, create PR (`/bitbucket create pr`), create snapshot (`/ios-snapshot`) for that repo too
     7. Go back to the main app repo
     8. Update the main app's Podfile to also point to this newly-fixed repo via local path
     9. Run `pod install` and build again
     10. **Repeat** if more dependent repos break — keep fixing cascading dependencies until the main app builds
   - If it does NOT exist locally: notify the user which repo needs to be cloned and adapted

## Step 5: Run CI test suite for each changed repo

After the main app builds successfully, run the CI test scheme for **every repo that was changed** (sub-module + any downstream repos with cascading fixes).

For each changed repo:
```bash
cd <repo-path>
xcworkspace=$(find . -maxdepth 1 -type d -name "*.xcworkspace" | head -1 | sed 's|^\./||')
moduleName=$(basename "$xcworkspace" .xcworkspace)
TEST_SCHEME="${moduleName}Tests"
echo "Running $TEST_SCHEME in $xcworkspace"

xcodebuild test -workspace "$xcworkspace" -scheme "$TEST_SCHEME" \
  -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16' test
```

Report: `X tests, Y failures` for each repo.

## Step 6: Generate manual test cases (MANDATORY — must be clearly visible output)

After tests pass, generate test cases for self-verification. **These MUST be output as a distinct, clearly labeled section** — not buried in workflow logs.

Based on the ticket requirements and code changes, create a list of manual test scenarios:
- **Happy path**: the main flow works as expected with the new changes
- **Edge cases**: boundary conditions, empty states, error states
- **Regression**: existing flows that touch the same code still work
- **Country-specific**: if the change is country-specific, note which country to test on

Format as a clearly labeled block:
```
=== Manual Test Cases (PNL-XXXXX) ===

TC-1: {Title}
- Precondition: {setup needed}
- Steps: {what to do}
- Expected: {what should happen}

TC-2: ...
```

**Note**: The main app verify step only **compiles** — it does NOT install or launch the app on the simulator. Manual testing requires running the app from Xcode on a simulator or device.

## Safety

- Only modify Podfile in repos listed in the ticket scope
- Only push to `feature/*` or `snapshot/*` branches on downstream repos
- Never force push or delete branches on repos you don't own
- When building main app, only build — never push or create PRs on the main app repo

## Step 7: Log output

```
=== [ios-verify] ===
Status: PASSED | FAILED
Main app: <repo name>
Build result: PASSED | FAILED
Cascading fixes: <list of repos fixed, or "none">
Cascading PRs: <PR links, or "none">
CI test suites: <repo>: X tests, Y failures | ...
Test cases: <count> generated
```
