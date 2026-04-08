# Android: Implement presentation changes for a ticket

$ARGUMENTS

This skill handles the **Android-specific** implementation steps. Shared steps (requirement confirmation, review gate) are in `/mobile-clarify`.
For design system component mapping, use `/android-cds`.
For KMP shared logic changes, use `/kmp-implement`.

## Prerequisites

Run `/mobile-clarify` first to complete:
- Requirement understanding and confirmation (Step 1 + 1b)
- Figma design review (if UI ticket)

Only proceed here after the user has approved the requirement confirmation.

## KMP-Aware Implementation

When KMP changes were made (via `/kmp-implement`), this skill can start **immediately** without waiting for the KMP snapshot to finish publishing. Use your knowledge of the KMP changes (new types, APIs, models, use cases) to write the Android code.

- **During implementation (Steps 1-4)**: Write code based on the KMP types/APIs you know were created. You have context from the KMP implementation about what was added/changed.
- **During local verify (Step 5)**: The KMP dev version is needed to resolve dependencies. If not yet published to local Maven, **ask the user for the KMP version** before proceeding with Step 5.

## Step 1: Research

Before writing any code:
1. Search for existing classes/models/ViewModels mentioned in the ticket: `Glob **/<Name>.kt`
2. Check `gradle/libsCustom.versions.toml` for external dependencies — they already exist, NEVER recreate them
3. Read ALL files that will need changes
4. Identify the architecture pattern used (MVVM, Hilt DI, RemoteConfig, etc.)
5. Check the module structure: `find . -name "build.gradle.kts" -path "*/feature/*"`
6. **If the ticket involves KMP shared version changes**: sync Gradle first, then verify new KMP types are available by checking imports resolve

## Step 1b: Design system mapping (if UI ticket) → `/android-cds`

If the ticket involves UI changes, run `/android-cds` to map Figma components to their implementations in `tc-mx-android-design-system`. This ensures correct component usage before writing any UI code. Do NOT improvise DS components — if a component isn't found, `/android-cds` will ask for clarification.

## Step 2: Plan

- List each file to modify and what changes
- For UI changes: reference the `/android-cds` mapping to confirm which DS components to use
- Confirm scope: which flows to touch, which to leave alone
- If adding new dependencies: trace the DI path (Hilt module → @Provides / @Binds → injection site)
- **If touching multiple repos**: sync Gradle in ALL affected repos before building — stale dependency caches cause phantom "unresolved reference" errors
- Present the plan and ask for confirmation

## Step 3: Implement

- Follow existing patterns in the codebase
- Don't modify flows/methods not related to the ticket
- Don't add fields to existing models unless explicitly required
- Refactor large methods into focused private methods
- If new files are needed, check that they don't already exist first
- **CLAUDE.md**: Many repos gitignore `.claude/` — check `.gitignore` before trying to commit `.claude/CLAUDE.md`. If gitignored, CLAUDE.md is local-only guidance for Claude Code.

### Android-specific conventions

#### Feature Toggles

- Boolean: `FeatureToggle` enum with `key`, `usage`, `defaultValue`
- JSON config: `JSONObjectFeatureFlag` enum with `key`, `usage`, `defaultJsonFileName`
- When adding new flag entries, always update the corresponding `*FlagTest.kt`
- Place default JSON in ALL country-specific local config modules (`phLocalConfig`, `saLocalConfig`) under `src/main/assets/`

#### Model Conventions

- **Remote Config Model**: `@Serializable` data class, all fields with default values, `@SerialName` annotations
- **UI Model**: data class with `companion object { fun from() }` factory (NOT extension functions), `ImmutableList` for list fields, `resolveDrawable: (String) -> Int` lambda for drawable resolution. Do NOT add `@Immutable` annotation on data classes.

#### ViewModel Pattern

- `@HiltViewModel` with `@Inject constructor`
- `remoteConfigManager` param is NOT private (linter enforces)
- Use `collectInScope` for flow collection, `MutableStateFlow` + `asStateFlow()` for state
- SharedPreferences: access directly via `context.getSharedPreferences()` (no DI injection needed)

#### Compose UI

- Screen composable (stateful, `internal`): collects state, delegates to content composable
- Content composable (stateless, `private`): pure UI, no ViewModel reference
- Self-contained feature screens: accept ViewModel directly, own visibility state, animate with `AnimatedVisibility`
- Use `remember(key1, key2)` for lambda stability when depending on boolean flags
- DS components: `TopNavigation2`, `PrimaryButton2`, `ButtonDock2`, `noRippleClickable`, `singleClick`
- Resource alias: `import com.tyme.digital.androidResources.R as TxResources`

#### Activity Pattern

- Extends `TimeoutActivity` (not `AppCompatActivity`)
- `enableEdgeToEdge()` in `onCreate`
- Pass ViewModels explicitly to screen composables via `by viewModels()`

#### Resource Keep (R8 shrinking)

```xml
<!-- res/raw/keep.xml -->
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@drawable/specific_name_1,@drawable/specific_name_2" />
```
Only list specific drawable names referenced dynamically. NO wildcards.

### Version bumping (MANDATORY)

- Format: `MAJOR.MINOR.PATCH` in each module's `build.gradle.kts`
- Bump the **minor** version per ticket (e.g., 1.5.0 → 1.6.0)
- Bump ALL affected modules (presentation + local configs if assets changed)
- Before bumping, compare against `master`: `git diff master -- build.gradle.kts`. If already bumped on this branch, skip.
- This applies to EVERY ticket — even small changes need a version bump for the snapshot to publish a new version.

## Step 4: Tests

- Use **MockK** for mocking, **Turbine** for flow testing (`flow.test { awaitItem() }`)
- Test naming: `` `methodName WHEN condition THEN expected behavior`() = runTest { ... } ``
- Group tests with section headers: `// ========== featureName Tests ==========`
- Add tests for each new code path: ViewModel, serialization model, UI model factory, flag enum
- Mock remote config: `every { remoteConfigManager.subscribe<Boolean>(flag) } returns flowOf(true)`
- For JSON config: use `emptyFlow()` to avoid `android.util.Log` mock issues with `safeDrawable`
- Serialization tests: verify encode/decode roundtrip and default values with `Json { ignoreUnknownKeys = true }`
- UI model tests: test `from()` factory with mock `resolveDrawable` lambda

## Step 5: Local verify (mandatory — do NOT skip or stop early)

### 5-pre. KMP local Maven integration (if KMP changes were involved)

If KMP changes were made (via `/kmp-implement`), the presentation repo needs the KMP dev version published to local Maven before it can build.

1. **Locate the KMP sibling repo** — replace `android` with `kmp` in the repo directory name:
   ```bash
   ANDROID_REPO=$(basename "$(pwd)")
   KMP_REPO=$(echo "$ANDROID_REPO" | sed 's/android/kmp/')
   KMP_PATH="$(dirname "$(pwd)")/$KMP_REPO"
   echo "KMP repo: $KMP_PATH"
   ```

2. **Check if the KMP dev version is already in local Maven**:
   ```bash
   KMP_VERSION=$(grep -oP '(?<=version\s=\s")[^"]+' "$KMP_PATH"/*/build.gradle.kts | head -1)
   DEV_VERSION="${KMP_VERSION%-*}-dev-claude"
   find ~/.m2/repository -path "*${DEV_VERSION}*" -name "*.pom" | head -1
   ```

3. **If NOT found in local Maven** — publish it automatically:
   ```bash
   cd "$KMP_PATH"
   # Set dev version in affected modules
   # Identify the module(s) that were changed
   KMP_MODULE=$(find . -maxdepth 2 -name "build.gradle.kts" -path "*/src/../*" -o -name "build.gradle.kts" ! -path "./build.gradle.kts" ! -path "*/buildSrc/*" | head -1 | xargs dirname | sed 's|^\./||' | tr '/' ':')
   ./gradlew :${KMP_MODULE}:publishToMavenLocal
   cd -
   ```

4. **Update the presentation repo** to consume the dev version:
   - Update KMP dependency version in `gradle/libsCustom.versions.toml` to the dev version (e.g., `X.Y.Z-dev-claude`)
   - Ensure `mavenLocal()` is in the repository list in `settings.gradle.kts`
   - Sync Gradle

5. **Verify KMP types are available**: check that new imports resolve without errors by running a quick build:
   ```bash
   ./gradlew :feature:${FEATURE_MODULE}:compileDebugKotlin
   ```

If the compile succeeds, KMP integration is verified. Proceed with the remaining verify steps below.

> **Remember**: dev versions are temporary. They will be reverted to official versions before final commit (see Version bumping section).

Run ALL of the following checks in order. Do not stop until every check is done, even if one fails (report all results at end).

### 5a. Detect the feature module
```bash
FEATURE_MODULE=$(find . -maxdepth 2 -path "*/feature/*/build.gradle.kts" | head -1 | sed 's|./feature/||;s|/build.gradle.kts||')
echo "Feature module: $FEATURE_MODULE"
```
**Use this `FEATURE_MODULE` for all build/test commands. Do NOT guess or hardcode module names.**

### 5b. Spotless formatting
```bash
./gradlew spotlessApply
```

### 5c. Build and run unit tests (primary repo)
```bash
./gradlew :feature:${FEATURE_MODULE}:testDebugUnitTest
```

### 5d. If touching multiple repos (e.g., shared module + downstream consumer):
For EACH affected downstream repo:
1. Update `gradle/libsCustom.versions.toml` to use the changed module's dev version
2. **If KMP shared version differs**: also update KMP version in `libsCustom.versions.toml` to match
3. Sync Gradle
4. Run `./gradlew :feature:{module}:testDebugUnitTest`
5. **After verifying**, revert `libsCustom.versions.toml` changes (they're temporary for local testing)

### 5e. Report all results before proceeding

## Log output

After completing all steps, present the **Implementation Review Gate** (from `/mobile-clarify`): summary of all changes, wait for user approval.

```
=== [android-implement] ===
Status: PASSED | FAILED
Requirement: <1-line summary>
Country target: <GTZA/SLZA/GTPH/unchanged>
Files changed: <count and list>
Files created: <count and list>
Tests added: <count and list>
```

## Next steps
- Run `/android-precheck` to validate before PR (only after user approves implementation)
- Run `/bitbucket create pr` to commit and open PR
- Run `/mobile-snapshot` to publish snapshot version
- Run `/android-verify` to verify on main app

$ARGUMENTS
