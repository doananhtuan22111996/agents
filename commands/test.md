# /test — Test Strategy & Test Plan
 
Create a test strategy and test plan for: **$ARGUMENTS**
 
## Instructions
 
Produce a test plan covering all relevant test types for this feature. Focus on what actually matters — critical paths, failure modes, and regression risks.
 
---
 
**Title**: [Project] Test Plan — [Feature Name]
**Created**: [today's date]
 
---
 
## Test Scope
What is being tested? What is explicitly NOT being tested in this plan?
 
## Test Strategy
 
### Unit Tests
What to test at the unit level:
- ViewModels: state transitions, event handling, error cases
- Use Cases / business logic
- Utility functions with meaningful logic
- Repository logic (with mocked data sources)
 
**Framework**: JUnit4/5 + MockK + Turbine (for Flow)
 
### Integration Tests
What needs integration-level testing (if any):
- Repository + Room DAO integration
- Repository + Retrofit integration (MockWebServer)
 
**Framework**: JUnit4 + Room in-memory DB + MockWebServer
 
### UI Tests (Compose)
Screens or flows that need UI-level testing:
- Critical user flows (e.g., create task, mark done)
- Error state rendering
- Empty state rendering
 
**Framework**: Compose Test Rule + `composeTestRule.onNode(...)`
 
### Manual Test Scenarios
Test cases that require human judgment or device interaction:
 
| # | Scenario | Steps | Expected Result | Status |
|---|----------|-------|-----------------|--------|
| 1 | Happy path | ... | ... | [ ] |
| 2 | Empty state | ... | ... | [ ] |
| 3 | Error state | ... | ... | [ ] |
| 4 | Offline | ... | ... | [ ] |
| 5 | Orientation change | ... | ... | [ ] |
| 6 | Process death | ... | ... | [ ] |
 
### Regression Risks
List of existing features that could be broken by this change. Verify these still work.
 
---
 
## Unit Test Examples
Write actual test skeletons for the most critical cases:
 
```kotlin
@Test
fun `when X happens, state should be Y`() = runTest {
    // Arrange
 
    // Act
 
    // Assert
}
```
 
---
 
## Acceptance Criteria Verification
Map each acceptance criterion from the PRD to a specific test case. Ensures full coverage.
 
| AC | Test Type | Test Name/Scenario |
|----|-----------|-------------------|
| AC-01 | Unit | ... |
| AC-02 | Manual | ... |
