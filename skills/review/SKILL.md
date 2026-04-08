# /review — Self Code Review
 
Review the current code changes (or: $ARGUMENTS) against the full checklist.
 
## Instructions
 
Run `git diff HEAD` or inspect the specified files. Be critical — the goal is to catch issues before opening a PR.
 
---
 
## ✅ Correctness
- [ ] Logic is correct for all inputs: empty, null, boundary, concurrent
- [ ] State is updated atomically where needed (no partial update bugs)
- [ ] `StateFlow` / `SharedFlow` emissions are correct — no missed events, no hot/cold confusion
- [ ] `suspend` functions called from the correct coroutine scope and dispatcher
- [ ] `collect` / `collectLatest` chosen appropriately (cancellation behavior)
- [ ] No race conditions in async operations
 
## 🔒 Security
- [ ] No tokens, PII, or passwords in logs (`Log.d`, Timber, or Crashlytics)
- [ ] No sensitive data in error messages or API responses surfaced to the UI
- [ ] Auth/permission gate in place before accessing protected data or screens
- [ ] Inputs validated and sanitized before use (especially from deep links / Intents)
- [ ] No hardcoded API keys or secrets
- [ ] Sensitive prefs use `EncryptedSharedPreferences`, not plain `SharedPreferences`
- [ ] Deep link / Intent extras treated as untrusted input
 
## ⚡ Performance
- [ ] No unnecessary Compose recompositions — `remember`, `derivedStateOf`, stable keys used correctly
- [ ] `LazyColumn` / `LazyRow` items use stable keys (`key = { item.id }`)
- [ ] No N+1 Room queries — use JOIN or batch fetch
- [ ] Heavy work on `Dispatchers.IO`, never on `Main`
- [ ] No memory leaks: `ViewModel` scope, `lifecycleScope`, no static `Context` refs
- [ ] Bitmaps and large objects released when no longer needed (Coil handles most, but custom code must too)
- [ ] No `runBlocking` on the main thread
 
## 🛡️ Error Handling
- [ ] Network errors caught and mapped to UI state (not a crash)
- [ ] Room/DB errors handled (DAO can throw, wrap in try/catch or `Result`)
- [ ] Unexpected exceptions caught at ViewModel or repository boundary — not silently swallowed
- [ ] User sees a meaningful error message, not a raw exception string
- [ ] Retry / offline fallback logic in place where needed
- [ ] `Result<T>` or sealed `UiState` used to propagate errors cleanly
 
## 🧪 Tests
- [ ] ViewModel state transitions unit-tested with `TestCoroutineDispatcher` / `runTest`
- [ ] Use case / business logic covered with JUnit + MockK
- [ ] Flows tested with Turbine (`test { }`)
- [ ] No untested `!!` force-unwraps on production paths
- [ ] Tests assert behavior, not implementation details
 
## 🧹 Code Quality (Kotlin)
- [ ] No `!!` without an explicit comment justifying it
- [ ] `?.let`, `?.run`, `when` used idiomatically — not chained into unreadable nests
- [ ] No unused variables, imports, or dead code
- [ ] Extension functions placed in the correct file (not bloating unrelated classes)
- [ ] No TODO without a linked Notion task
- [ ] Naming is clear: no `data2`, `temp`, `result2`, `manager2`
 
## 🏗️ Architecture
- [ ] No business logic in `@Composable` functions — belongs in ViewModel or UseCase
- [ ] No Android types (`Context`, `Intent`, `Resources`) in domain/use-case layer
- [ ] Repository correctly abstracts data sources (remote vs local)
- [ ] ViewModel does NOT hold `Context` or `Activity` references
- [ ] Navigation events emitted as one-shot `Channel` / `SharedFlow`, not `StateFlow`
- [ ] Single source of truth — UI observes one state, not multiple conflicting sources
 
## 📱 Android Platform
- [ ] Survives configuration change (rotation, dark mode, font scale) — ViewModel saves state
- [ ] Survives process death — `SavedStateHandle` used for critical transient state
- [ ] Runtime permissions handled correctly: request → granted/denied → handle denied gracefully
- [ ] Back gesture / predictive back handled (`BackHandler` if needed)
- [ ] Works offline or degrades gracefully with a clear message
- [ ] Tested on API min target, not just latest
 
---
 
## Summary
 
**Issues Found**
- 🔴 Must Fix: ...
- 🟡 Should Fix: ...
- 🟢 Nice to Have: ...
 
**Overall**: Ready to merge | Needs changes | Significant rework needed
