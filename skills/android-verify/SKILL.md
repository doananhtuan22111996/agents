# Android: Verify sub-module changes on main app

$ARGUMENTS

Verify that sub-module changes integrate correctly with the main app, fix cascading dependency issues, and generate manual test cases.

## Step 1: Identify the main app repo

Based on the country target from the ticket:

| Target | Main app repo |
|--------|--------------|
| GTZA | `tb-gosa-android-app` |
| SLZA | `tb-sanlam-android-app` |
| GTPH | `ph-goph-android-app` |

If no country was specified, ask the user which main app to verify against.

## Step 2: Update main app to use local module

In the main app repo, update the dependency to point to the changed sub-module's local path:

- In `gradle/libsCustom.versions.toml`, change the sub-module version to the dev version (e.g., `X.Y.Z-dev-claude`)
- If the sub-module was published to local Maven (`./gradlew :{module}:publishToMavenLocal`), ensure `mavenLocal()` is in the repository list in `settings.gradle.kts` or root `build.gradle.kts`
- **If KMP shared version was updated in the sub-module**: also update `shared` version in `gradle/libsCustom.versions.toml` to match. Mismatched shared versions cause "unresolved reference" build errors.
- Sync Gradle to resolve the local changes

## Step 3: Build the main app

The main app uses a multi-dimension flavor matrix: `{Environment}{Mode}{Market}{BuildType}`.

```bash
./gradlew :androidApp:app:assembleUatManualGoogleDebug
```

Common variants:
| Variant | Use case |
|---------|----------|
| `assembleUatManualGoogleDebug` | Default for verification (UAT + Manual + Google + Debug) |
| `assembleSitManualGoogleDebug` | SIT environment |
| `assembleDevManualGoogleDebug` | DEV environment |
| `assembleMockManualGoogleDebug` | Mock environment (offline) |

If the app module path differs from `:androidApp:app`, check `settings.gradle.kts` for the correct module path.

## Step 4: Handle build result

### If build SUCCEEDS → go to Step 5

### If build FAILS → analyze errors

1. **Failure is unrelated to sub-module changes** → Note the pre-existing failure and confirm sub-module changes are not the cause. Go to Step 5.

2. **Failure is caused by sub-module changes breaking another dependent repo** (e.g., sub-module API changed, and an intermediate module doesn't compile):
   - Identify which dependent repo is broken (check the error: which module/file fails)
   - Check if the source for that repo exists locally (e.g., `../tc-mx-android-payment`)
   - If it exists locally:
     1. Jump into that repo
     2. Update its `gradle/libsCustom.versions.toml` to use the changed sub-module's dev version
     3. Sync Gradle
     4. Implement the necessary adaptations to make it compile (follow same patterns as `/android-implement`)
     5. Run `/android-precheck` on that repo
     6. Commit, create PR (`/bitbucket create pr`), create snapshot (`/mobile-snapshot`) for that repo too
     7. Go back to the main app repo
     8. Update the main app's dependency to also point to this newly-fixed repo
     9. Sync Gradle and build again
     10. **Repeat** if more dependent repos break — keep fixing cascading dependencies until the main app builds
   - If it does NOT exist locally: notify the user which repo needs to be cloned and adapted

## Step 5: Run CI test suite for each changed repo

After the main app builds successfully, run the CI test scheme for **every repo that was changed** (sub-module + any downstream repos with cascading fixes).

For each changed repo:
```bash
cd <repo-path>
./gradlew :feature:{moduleName}:testDebugUnitTest
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

**Note**: The main app verify step only **compiles** — it does NOT install or launch the app on an emulator. Manual testing requires running the app from Android Studio on an emulator or device.

## Safety

- Only modify dependency versions in repos listed in the ticket scope
- Only push to `feature/*` or `snapshot/*` branches on downstream repos
- Never force push or delete branches on repos you don't own
- When building main app, only build — never push or create PRs on the main app repo

## Step 7: Log output

```
=== [android-verify] ===
Status: PASSED | FAILED
Main app: <repo name>
Build result: PASSED | FAILED
Cascading fixes: <list of repos fixed, or "none">
Cascading PRs: <PR links, or "none">
CI test suites: <repo>: X tests, Y failures | ...
Test cases: <count> generated
```
