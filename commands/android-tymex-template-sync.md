<!--
  Approach-2 LAUNCHER — android-tymex-template-sync (Android only)
  Installed to: ~/.claude/commands/android-tymex-template-sync.md
  Assets source: https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/
  Required env vars: BITBUCKET_USERNAME  BITBUCKET_PASSWORD
  Canonical (relative-path) version: android-tymex-template-sync/SKILL.md
  Platform: Android
-->

---
name: android-tymex-template-sync
description: >
  Aligns a TymeX Android feature repository with the latest
  tc-mx-android-feature-template standard. Use this skill whenever someone
  says: "align with template", "apply template changes", "sync build config
  with template", "CB-11440/CB-11442/CB-11454 style changes", "separate
  libsCoreExample from libsCustom", "bump mobile-versions BOM", or
  "template alignment PR". Detects the repo's current state and applies
  only the missing changes from 8 well-defined categories. Safe to run on
  partially-aligned repos — already-done changes are never re-applied.
---

# TymeX Android Template Sync Skill (Android only)

Aligns any TymeX Android feature repository with the build-infrastructure
standard defined in `tc-mx-android-feature-template`.

**Source of truth:**
`https://bitbucket.org/tymerepos/tc-mx-android-feature-template/src/master/`

---

## Prerequisites

Before running, confirm:
- You are inside the root of the target TymeX feature repository.
- The repo follows the expected module layout:
  `feature/<FeatureNamePresentation>/` and `exampleApp/`.
- `CODEARTIFACT_AUTH_TOKEN` env var is set, or `local.properties` contains it.

---

## Step 1 — DETECT

Run the detection script from the repo root:

```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/scripts/detect_state.py" \
  -o /tmp/tymex_detect_state.py
python3 /tmp/tymex_detect_state.py .
```

The script prints a JSON gap report. Each of 8 categories has status:
- `OK` — already aligned, skip
- `PARTIAL` — partially done, see details
- `MISSING` — needs full application

Example output:
```json
{
  "A_catalog_separation": { "status": "MISSING", "details": [...] },
  "B_bom_bump":           { "status": "PARTIAL", "current": "2.50.0", "target": "2.83.0" },
  "C_exampleapp_build":   { "status": "MISSING", "details": [...] },
  "D_dep_substitution":   { "status": "OK" },
  "E_exampleapp_code":    { "status": "MISSING", "details": [...] },
  "F_flavors":            { "status": "OK" },
  "G_test_rule":          { "status": "MISSING", "details": [...] },
  "H_buildsrc_kotlin":    { "status": "OK" }
}
```

---

## Step 2 — PLAN

Present the gap report to the user as a plain table, e.g.:

```
Category  Status   Description
────────────────────────────────────────────────────────────────────────
A         MISSING  Version catalog separation (libsCoreExample / libsCustom)
B         PARTIAL  BOM version bump (2.50.0 → 2.83.0)
C         MISSING  ExampleApp build.gradle.kts alignment
D         OK       Root dependency substitution
E         MISSING  ExampleApp code restructure (MyApplication + Module)
F         OK       Multi-flavor resources (mock/sit/uat)
G         MISSING  TestCoroutineRule utility class
H         OK       buildSrc Kotlin source files
```

Ask: **"I'll apply categories A, B, C, E, G. Skip D, F, H which are already aligned.
Proceed?"**

Wait for user confirmation before executing.

---

## Step 3 — EXECUTE

Apply categories **in order A → H**. Skip any that are `OK`.

Fetch change-categories for exact before/after rules (at runtime):
```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/references/change-categories.md"
```

Fetch file-templates for full file content to create (at runtime):
```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/references/file-templates.md"
```

### Category A — Version Catalog Separation

1. **CREATE** `gradle/libsCoreExample.versions.toml`
   Copy exact content from the `libsCoreExample` section from file-templates (fetch at runtime).
```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/references/file-templates.md"
```

2. **CLEAN** `gradle/libsCustom.versions.toml`
   Remove these keys if present (they move to libsCoreExample):
   `mobilePlugins`, `slackSupport`, `exampleTymeX`, `phAndroidConfiguration`,
   `phLaunchDarklyCredential`, `phAndroidResources`, `gophAppHeaderPresentation`,
   `tymeXCommonPopupProvider`.
   **Never remove** feature-specific entries (the repo's own business library).

3. **UPDATE** `buildSrc/settings.gradle.kts`
   Replace `create("libsCustom")` block with `create("libsCoreExample")` block.
   Exact content: see the `buildSrc/settings.gradle.kts` section from file-templates (fetch at runtime).
```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/references/file-templates.md"
```

4. **UPDATE** `buildSrc/build.gradle.kts`
   Change `implementation(libsCustom.mobilePlugins)` →
   `implementation(libsCoreExample.mobilePlugins)`.

### Category B — BOM Version Bump

In `settings.gradle.kts`:
1. Change `mobile-versions:X.Y.Z` → `mobile-versions:2.83.0`
2. Add `libsCoreExample` catalog entry after the `libs` catalog block:
   ```kotlin
   create("libsCoreExample") {
       from(files("gradle/libsCoreExample.versions.toml"))
   }
   ```
   (only if not already present)

### Category C — ExampleApp Build Config

In `exampleApp/build.gradle.kts`:
1. `tymeXFlavors(project)` → `tymeXFlavors(project, true)`
   (if the call has no second arg, add `, true`)
2. Add `addImplementFromLibsDependencies(libsCoreExample)` to `dependencies {}`
   block (after the existing `addImplementFromLibsDependencies(libsCustom)` line).
3. Ensure `implementation(libs.tymexNavigation)` is present in `dependencies {}`.

### Category D — Root Dependency Substitution

In `build.gradle.kts` (root), inside `subprojects { configurations.all { ... } }`:
Add the `credentials-api` substitution block after the `router` one:
```kotlin
if (requested.module == "credentials-api") {
    val dep = libs.kmmCoreApi.get()
    val expected = "${dep.module.group}:${dep.module.name}:${dep.versionConstraint.requiredVersion}"
    useTarget(expected)
}
```
Only add if a `subprojects { configurations.all { resolutionStrategy... } }` block
exists and the `credentials-api` check is absent.

### Category E — ExampleApp Code Restructure

1. **CREATE** `exampleApp/src/main/java/com/tymex/app/MyApplication.kt`
   if it does not exist or is not in the `com.tymex.app` package.
   Content: see the `MyApplication.kt` section from file-templates (fetch at runtime).
```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/references/file-templates.md"
```

2. **CREATE** `exampleApp/src/main/java/com/tymex/di/ExampleAppModule.kt`
   if it does not exist or the import uses the old router API.
   Content: see the `ExampleAppModule.kt` section from file-templates (fetch at runtime).
```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/references/file-templates.md"
```
   ⚠ Preserve the existing `PATH_*` constant and deeplink string — only
   update the import from `router.in_app.navigateTo` → `navigation.api.navigateTo`.

3. **REMOVE** old package files if present:
   - `exampleApp/src/main/java/tymex/app/MyApplication.kt` (old package)
   - `exampleApp/src/main/java/tymex/app/AppLifecycleListener.kt`
   - `exampleApp/src/main/java/tymex/app/di/` (old DI folder)

4. **UPDATE** `exampleApp/src/main/AndroidManifest.xml`
   Ensure `android:name=".app.MyApplication"` is set on `<application>`.

### Category F — Multi-Flavor Resources

For each flavor in `[mock, sit, uat]`:
- If `exampleApp/src/<flavor>/` does not exist, create it with:
  - `google-services.json` — see the `google-services.json` section from file-templates (fetch at runtime):
```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/references/file-templates.md"
```
  - `res/values/strings.xml` — see the `strings.xml` section from file-templates (fetch at runtime):
```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/references/file-templates.md"
```
  - For `mock` flavor only: `res/xml/network_security_config.xml`
    see the `network_security_config.xml` section from file-templates (fetch at runtime):
```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/references/file-templates.md"
```

### Category G — Test Coroutine Rule

Find the feature module's test source set:
`feature/<featureModule>/src/test/kotlin/<package>/`

Create the util directory and file:
`<above_path>/util/TestCoroutineRule.kt`
Content: see the `TestCoroutineRule.kt` section from file-templates (fetch at runtime).
```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/references/file-templates.md"
```
Only create if the file does not exist anywhere under `src/test/`.

### Category H — buildSrc Kotlin Source Files

1. **CHECK** `buildSrc/src/main/kotlin/Publishing.kt`
   - Must contain exactly: `const val GROUP_ID = "com.tymex"`
   - If missing: create it. If content differs: warn the user (do not overwrite).

2. **CHECK** `buildSrc/src/main/kotlin/BuildModules.kt`
   - Must follow the pattern: `object BuildModules { const val X = ":feature:..." }`
   - If file is missing: create a skeleton from the `BuildModules.kt` section in file-templates (fetch at runtime), then prompt the user to fill in their module path constants.
```bash
curl -s -u "$BITBUCKET_USERNAME:$BITBUCKET_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/tymerepos/tc-mx-mobile-claude-code-config/src/master/android-tymex-template-sync/references/file-templates.md"
```
   - If file exists but does not use `object BuildModules`: warn the user.
   - **Never auto-rewrite the constants** — they are feature-specific.

---

## Step 4 — VERIFY

Run the UAT assembly task from the repo root:

```bash
./gradlew :exampleApp:assembleUatManualGoogleDebug
```

- If **BUILD SUCCESSFUL**: all changes are valid. Report the categories applied.
- If **BUILD FAILED**: read the error output, identify the failing file/line,
  fix the issue, and re-run verification. Common failure causes:
  - Missing `google-services.json` for a flavor
  - Wrong package in `MyApplication.kt`
  - Import still using old `router.in_app` path
  - `libsCoreExample.versions.toml` has a typo in a library alias

---

## Safety Rules

1. **Never remove** lines from `libsCustom.versions.toml` that are NOT in the
   template standard set (the feature's own business libraries must stay).
2. **Never overwrite** `BuildModules.kt` constants — they encode module paths
   that are specific to the target repo.
3. **Do not apply** categories that are already `OK` — re-running is safe but
   must be truly idempotent (check before changing).
4. **Preserve** the feature-specific `PATH_*` constant and deeplink name in
   `ExampleAppModule.kt` when restructuring.
5. **Do not bump** `mobile-versions` to a version lower than the current one.
