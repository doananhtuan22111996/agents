# /deps — Gradle Dependency Management
 
Manage dependencies for: **$ARGUMENTS**
(e.g., "add Retrofit", "upgrade all libs", "audit unused deps", "check for updates")
 
## Instructions
 
Work with the project's `libs.versions.toml` (version catalog). Always use the catalog — never hardcode versions in `build.gradle.kts`.
 
---
 
## Audit Current State
 
First, inspect:
```bash
# Check for outdated deps
./gradlew dependencyUpdates
 
# Check for unused deps (if dependency-analysis plugin installed)
./gradlew analyzeDependencies
 
# Full dependency tree
./gradlew :app:dependencies --configuration releaseRuntimeClasspath
```
 
---
 
## Add a New Dependency
 
**Step 1 — Check before adding**
- Is there already a dep in the catalog that covers this?
- Is this an official Jetpack lib or a third-party one? Prefer Jetpack when available.
- Check current version at: https://maven.google.com or https://mvnrepository.com
 
**Step 2 — Add to `libs.versions.toml`**
```toml
[versions]
retrofit = "2.11.0"
 
[libraries]
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-converter-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "retrofit" }
 
[plugins]
# only if adding a Gradle plugin
```
 
**Step 3 — Reference in `build.gradle.kts`**
```kotlin
dependencies {
    implementation(libs.retrofit)
    implementation(libs.retrofit.converter.gson)
}
```
 
---
 
## Upgrade Dependencies
 
For each dep to upgrade:
1. Check release notes / changelog — any breaking changes?
2. Update version in `libs.versions.toml` only
3. Sync Gradle
4. Build: `./gradlew assembleDebug`
5. Run tests: `./gradlew test`
6. Check for deprecation warnings in the IDE
 
**Upgrade priority** (do in this order to minimize conflict):
1. Kotlin + KSP (must match)
2. AGP (Android Gradle Plugin)
3. Compose BOM (updates all Compose libs together)
4. Hilt (Hilt + KSP version must be compatible)
5. Room (Room + KSP version must be compatible)
6. Other Jetpack libs
7. Third-party libs
 
---
 
## Compose BOM Usage
Always use the BOM — never set individual Compose versions:
```toml
[versions]
compose-bom = "2024.09.00"
 
[libraries]
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }  # no version needed
compose-material3 = { group = "androidx.compose.material3", name = "material3" }
```
 
```kotlin
dependencies {
    implementation(platform(libs.compose.bom))
    implementation(libs.compose.ui)
    implementation(libs.compose.material3)
}
```
 
---
 
## Remove a Dependency
1. Remove all usages in code first
2. Remove from `build.gradle.kts`
3. Remove from `libs.versions.toml` (version + library entry)
4. Sync + build + test
 
---
 
## Security Check
```bash
# Check for known vulnerabilities (if OWASP dep check plugin installed)
./gradlew dependencyCheckAnalyze
```
Flag any dependency with a known CVE for immediate upgrade.
 
---
 
## Output
Produce a summary of changes made:
| Action | Library | Old Version | New Version | Notes |
|--------|---------|-------------|-------------|-------|
| Added | retrofit | — | 2.11.0 | For API layer |
| Upgraded | hilt | 2.50 | 2.51 | No breaking changes |
| Removed | gson | 2.10.1 | — | Replaced by kotlinx-serialization |
