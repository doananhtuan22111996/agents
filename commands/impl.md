# /impl — Implementation Plan
 
Create an implementation plan for: **$ARGUMENTS**
 
## Instructions
 
Explore the codebase first. Understand existing patterns before writing a single line. Then produce the plan — implement only when the plan is clear.
 
---
 
## 1. Requirement Check
- What exactly needs to be built? (restate concisely)
- Acceptance criteria from the PRD?
- What is explicitly out of scope for this task?
 
## 2. Codebase Exploration
Before planning, identify:
- Which existing feature is most similar? Use it as a pattern reference.
- Which files/classes will be created vs modified?
- Which Gradle modules are involved? (`:app`, `:data`, `:domain`, `:ui`, etc.)
- Any existing dependencies that cover this, or new ones needed in `libs.versions.toml`?
- Any database schema changes needed? (Room migration required?)
 
## 3. Architecture Layer Plan
 
Map the work to each layer:
 
```
UI Layer
  └── Screen: [XScreen.kt] — Composable, observes ViewModel state
  └── ViewModel: [XViewModel.kt] — holds UiState, handles UiEvent
  └── UiState: sealed class or data class
 
Domain Layer
  └── UseCase: [XUseCase.kt] — single responsibility, pure Kotlin
  └── Model: [X.kt] — domain model, no Android deps
 
Data Layer
  └── Repository interface: [XRepository.kt] in domain
  └── Repository impl: [XRepositoryImpl.kt] in data
  └── Remote: [XApiService.kt] — Retrofit interface + DTOs
  └── Local: [XDao.kt] + [XEntity.kt] — Room
  └── Mapper: entity/dto → domain model
```
 
## 4. Data Flow
```
User action
  → ViewModel.onEvent(UiEvent)
    → UseCase.execute(params)
      → Repository.getData()
        → Local (Room) or Remote (Retrofit)
  → UiState updated
    → Composable recomposes
```
 
## 5. State Design
Define the `UiState` shape upfront:
```kotlin
data class XUiState(
    val isLoading: Boolean = false,
    val items: List<X> = emptyList(),
    val error: String? = null
)
```
 
## 6. File Plan
| Action | File | Layer | Notes |
|--------|------|-------|-------|
| Create | `XViewModel.kt` | UI | |
| Create | `XScreen.kt` | UI | |
| Create | `XUseCase.kt` | Domain | |
| Create | `XRepository.kt` (interface) | Domain | |
| Create | `XRepositoryImpl.kt` | Data | |
| Create | `XDao.kt` | Data | Room migration needed? |
| Modify | `AppModule.kt` | DI | Hilt bindings |
| Modify | `NavGraph.kt` | Navigation | Add route |
 
## 7. Implementation Order
1. Domain model + Repository interface
2. Room entity + DAO (+ migration if schema changed)
3. Remote DTO + API service (if network involved)
4. Mapper (entity/DTO → domain)
5. Repository implementation
6. Use case
7. UiState + UiEvent definitions
8. ViewModel
9. Composable screen + components
10. Navigation wiring
11. Hilt bindings
12. Unit tests: ViewModel + UseCase
 
## 8. Edge Cases to Handle
- [ ] Empty state (no data yet)
- [ ] Loading state (skeleton or spinner)
- [ ] Error state (network failure, DB failure)
- [ ] Offline behavior (serve cache, show stale indicator?)
- [ ] Configuration change (ViewModel survives, UI re-subscribes)
- [ ] Process death (SavedStateHandle for critical state?)
 
## 9. Security Notes
- Any PII or sensitive data in this feature?
- Input from user/Intent that needs validation?
- Auth/permission gate required?
 
## 10. Quick Test Plan
- ViewModel: state transitions, error propagation
- UseCase: business rule correctness
- Manual: happy path + empty + error + offline
 
---
 
Proceed with implementation in the order above. Follow existing project conventions — check the nearest similar feature first.
