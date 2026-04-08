# /perf — Android Performance Audit
 
Run a performance audit on: **$ARGUMENTS**
(e.g., "the task list screen", "the whole app", "XViewModel", "Room queries in XRepository")
 
## Instructions
 
Systematic Android performance review. Don't optimize prematurely — measure first, then fix the top issues.
 
---
 
## Phase 1: Measure (before touching code)
 
### Compose Recomposition
```
Android Studio → Layout Inspector → Recomposition counts
Look for: any Composable recomposing more than expected per user action
```
 
### Rendering / Jank
```
adb shell dumpsys gfxinfo com.your.package
 
Target:
  - 90% frames < 16ms (60fps)
  - 99% frames < 32ms
  - Zero "Janky frames"
```
 
### App Startup
```
adb shell am start-activity -W -n com.your.package/.MainActivity
Look for: TotalTime > 500ms (cold start) or > 200ms (warm start)
```
 
### Memory
```
Android Studio → Memory Profiler
Profile: Heap dump, allocation tracking
Look for: growing heap over time (leak), large bitmaps, excessive allocations
```
 
### Database
```kotlin
// Enable Room query logging
Room.databaseBuilder(...)
    .setQueryCallback({ sqlQuery, bindArgs ->
        Timber.d("Room Query: $sqlQuery | Args: $bindArgs")
    }, Executors.newSingleThreadExecutor())
```
 
---
 
## Phase 2: Compose Performance Checklist
 
### Stability
- [ ] Data classes passed to Composables are `@Stable` or have all immutable fields
- [ ] Lambda parameters are stable — use `remember { { ... } }` for callbacks
- [ ] List types use `ImmutableList` (kotlinx.collections.immutable) to be recognized as stable
- [ ] `@Composable` functions that don't need to recompose are marked `@NonRestartableComposable`
 
### Keys & Derivation
- [ ] `LazyColumn` / `LazyRow` items all have `key = { item.id }` (prevents full rerender on insert)
- [ ] `remember(key)` uses the correct key — stale key = stale value
- [ ] `derivedStateOf { }` used when deriving computed state from another state to avoid over-recomposition
- [ ] `key()` composable used in conditional Composables to preserve state
 
### Side Effects
- [ ] `LaunchedEffect` uses specific keys — not `Unit` unless truly one-shot
- [ ] `SideEffect` not used for heavy work
- [ ] `DisposableEffect` properly cleans up (calls `onDispose`)
 
---
 
## Phase 3: Coroutines & Threading Checklist
- [ ] All IO (network, DB, file) runs on `Dispatchers.IO`
- [ ] No `runBlocking` on main thread
- [ ] `withContext(Dispatchers.Default)` for CPU-intensive work (sorting, mapping large lists)
- [ ] Flow operators: `flowOn(Dispatchers.IO)` at source, not downstream
- [ ] No blocking calls inside `suspend fun` without `withContext`
- [ ] `viewModelScope` used in ViewModel, `lifecycleScope` in Activity/Fragment
 
---
 
## Phase 4: Room / Database Checklist
- [ ] No query inside a loop — batch with `IN (:ids)` or JOIN
- [ ] Queries not on main thread — all DAO functions are `suspend` or return `Flow`
- [ ] Indexes on columns used in `WHERE` / `ORDER BY` — add `@ColumnInfo(index = true)` or explicit `@Index`
- [ ] `LIMIT` used on large queries
- [ ] `Flow` from Room auto-cancels when scope is cancelled — no manual cleanup needed
 
---
 
## Phase 5: Memory Checklist
- [ ] No `Context` or `Activity` stored in ViewModel or long-lived object
- [ ] Bitmaps not cached in memory beyond what Coil manages
- [ ] `WeakReference` used where needed for callbacks that outlive their owner
- [ ] No anonymous inner class referencing outer Activity (common leak)
- [ ] `LeakCanary` added to debug build and showing no leaks
 
---
 
## Phase 6: Startup Checklist
- [ ] Heavy init (DB creation, network setup) lazy or async — not in `Application.onCreate()`
- [ ] `App Startup` library (Jetpack) used if multiple initializers needed
- [ ] Hilt component initialization — no expensive work in `@Provides` methods called at startup
- [ ] First screen composable is lightweight (skeleton/shimmer, not blocking on data)
 
---
 
## Output
 
**Measurements** (fill in before and after):
| Metric | Before | After | Target |
|--------|--------|-------|--------|
| Cold start | ms | ms | < 500ms |
| Warm start | ms | ms | < 200ms |
| Janky frames | % | % | < 10% |
| Heap peak | MB | MB | < 100MB |
 
**Issues Found** (🔴 Critical / 🟡 Medium / 🟢 Minor):
- ...
 
**Changes Made**:
- ...
