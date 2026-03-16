# /test — Test Strategy & Test Plan
 
Create a test strategy and test plan for: **$ARGUMENTS**
 
## Instructions
 
Produce a full test plan for this feature. Focus on critical paths, failure modes, and regression risks. Write actual test skeletons — not just descriptions.
 
---
 
**Title**: [Project] Test Plan — [Feature Name]
**Created**: [today's date]
 
---
 
## Test Scope
- **In scope**: What is being tested?
- **Out of scope**: What is explicitly not tested here?
 
---
 
## Unit Tests
 
**Stack**: JUnit5 + MockK + Turbine + `kotlinx-coroutines-test`
 
### ViewModel Tests
```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class XViewModelTest {
 
    @get:Rule val mainDispatcherRule = MainDispatcherRule()
 
    private val useCase: XUseCase = mockk()
    private lateinit var viewModel: XViewModel
 
    @BeforeEach
    fun setup() {
        viewModel = XViewModel(useCase)
    }
 
    @Test
    fun `initial state is loading`() = runTest {
        viewModel.uiState.test {
            val state = awaitItem()
            assertThat(state.isLoading).isTrue()
        }
    }
 
    @Test
    fun `when use case succeeds, state contains items`() = runTest { ... }
 
    @Test
    fun `when use case throws, state contains error`() = runTest { ... }
 
    @Test
    fun `given offline, state shows cached data with stale flag`() = runTest { ... }
}
```
 
### UseCase Tests
```kotlin
class XUseCaseTest {
    private val repository: XRepository = mockk()
    private val useCase = XUseCase(repository)
 
    @Test
    fun `returns mapped domain model on success`() = runTest { ... }
 
    @Test
    fun `propagates repository exception as Result failure`() = runTest { ... }
}
```
 
### Repository Tests (with fakes or in-memory Room)
```kotlin
// Room: use in-memory DB
@RunWith(AndroidJUnit4::class)
class XRepositoryTest {
    private lateinit var db: AppDatabase
    private lateinit var repository: XRepositoryImpl
 
    @Before
    fun setup() {
        db = Room.inMemoryDatabaseBuilder(context, AppDatabase::class.java).build()
        repository = XRepositoryImpl(db.xDao(), mockk())
    }
}
```
 
---
 
## UI Tests (Compose)
 
**Stack**: `ComposeTestRule` + `@HiltAndroidTest` (if DI needed)
 
```kotlin
@HiltAndroidTest
class XScreenTest {
 
    @get:Rule val composeRule = createAndroidComposeRule<MainActivity>()
 
    @Test
    fun `shows empty state when no items`() {
        composeRule.setContent {
            XScreen(state = XUiState(items = emptyList()))
        }
        composeRule.onNodeWithText("No items yet").assertIsDisplayed()
    }
 
    @Test
    fun `shows error message on failure`() { ... }
 
    @Test
    fun `clicking item navigates to detail`() { ... }
}
```
 
---
 
## Manual Test Scenarios
 
| # | Scenario | Steps | Expected | Status |
|---|----------|-------|----------|--------|
| 1 | Happy path | ... | ... | [ ] |
| 2 | Empty state | No data in DB | Shows empty state UI | [ ] |
| 3 | Error state | Kill network, trigger load | Shows error + retry | [ ] |
| 4 | Offline | Airplane mode | Shows cached data | [ ] |
| 5 | Rotation | Mid-operation, rotate | State preserved | [ ] |
| 6 | Process death | Force stop, reopen | State restored | [ ] |
| 7 | Back gesture | Predictive back | Correct navigation | [ ] |
| 8 | Permission denied | Deny permission | Graceful fallback | [ ] |
 
---
 
## Regression Risks
Which existing features could break due to this change?
- ...
 
Verify manually after implementation.
 
---
 
## AC Coverage Map
| Acceptance Criterion | Test Type | Test Name |
|---------------------|-----------|-----------|
| AC-01 | Unit | `ViewModel: when X, state is Y` |
| AC-02 | Manual | Scenario 1 |
| AC-03 | UI | `shows empty state` |
