# /debug — Android Debug Session
 
Debug the following issue: **$ARGUMENTS**
 
## Instructions
 
Structured debugging for Android. Work through each phase systematically. Don't guess — confirm each hypothesis with evidence before moving on.
 
---
 
## Phase 1: Reproduce
 
**Goal**: Get a reliable reproduction before touching any code.
 
- Can you reproduce it every time? Or is it intermittent?
- What are the exact steps to trigger it?
- Which device / API level / screen size?
- Does it happen on fresh install? After specific user action? Only in release build?
 
Collect:
```bash
# Full logcat filtered to your app
adb logcat --pid=$(adb shell pidof -s com.your.package)
 
# Crash log if ANR / crash
adb pull /data/anr/traces.txt
```
 
---
 
## Phase 2: Locate
 
**Goal**: Narrow down WHERE in the code the problem originates.
 
Checklist:
- [ ] Which layer? UI / ViewModel / UseCase / Repository / Data source
- [ ] Check Logcat for stack trace — what's the first line in YOUR code (not framework)?
- [ ] Is it a state issue (wrong data shown) or a crash?
- [ ] Is it timing-related? (race condition, coroutine cancellation, lifecycle)
- [ ] Does it happen in debug but not release (or vice versa)? → Proguard / minification
 
Narrow down with targeted logs:
```kotlin
Timber.d("XViewModel: state = $uiState")
Timber.d("Repository: result = $result")
```
 
---
 
## Phase 3: Understand
 
**Goal**: Know exactly WHY it fails before writing a fix.
 
- What is the code expecting vs what it actually receives?
- Is the failure deterministic or dependent on timing / thread?
- Common Android-specific culprits:
  - **Lifecycle**: `collect` called after `onStop`, Activity recreated, back stack
  - **Coroutine scope**: cancelled scope, wrong dispatcher, `viewModelScope` vs `lifecycleScope`
  - **Compose**: recomposition trigger, `remember` key changed, `LaunchedEffect` re-fired
  - **Room**: migration missing, wrong thread, DAO not suspend
  - **State**: `StateFlow` vs `SharedFlow` confusion, stale state after config change
  - **Process death**: `SavedStateHandle` not used, ViewModel recreated
 
Reproduce the bug mentally: walk through the code path step by step.
 
---
 
## Phase 4: Fix
 
**Goal**: Smallest correct change that addresses the root cause, not the symptom.
 
- Don't patch the symptom — fix the root cause
- If you need to change 10+ files, question the approach
- Add a comment if the fix is non-obvious:
  ```kotlin
  // Fix: collectLatest cancels previous collection when screen rotates,
  // preventing stale emission from previous lifecycle
  ```
 
---
 
## Phase 5: Verify
 
After the fix:
- [ ] Reproduce original bug — does it still occur? No → good
- [ ] Run unit tests: `./gradlew test`
- [ ] Test the fixed scenario on the same device/API level
- [ ] Test edge cases: empty state, error state, offline, rotation
- [ ] Check logcat — no new warnings or errors introduced
- [ ] Run on min SDK target to catch API compatibility issues
 
---
 
## Phase 6: Document
 
If this was non-trivial:
- Add a comment in the code explaining WHY
- Use `/doc` to write a short learning note to Notion
- Consider adding a regression test so it never comes back
