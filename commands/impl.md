# /impl — Implementation Plan
 
Create an implementation plan for: **$ARGUMENTS**
 
## Instructions
 
Before writing any code, produce a clear implementation plan. This is the thinking phase. Then implement if asked.
 
---
 
## 1. Understand the Requirement
- What exactly needs to be built?
- What are the acceptance criteria?
- What is explicitly out of scope?
 
## 2. Explore Codebase First
Identify:
- Existing patterns in the project that should be followed
- Files that will need to be created or modified
- Dependencies already available vs. what needs to be added
- Potential conflicts with existing code
 
## 3. Approach
Describe the implementation approach in plain language:
- Architecture layer involvement (UI → ViewModel → UseCase → Repository → DataSource)
- Data flow: where data comes from, how it transforms, where it ends up
- State management approach
- Error handling strategy
 
## 4. File Plan
| Action | File/Class | Purpose |
|--------|-----------|---------|
| Create | `feature/X/XViewModel.kt` | Manages UI state for X |
| Create | `feature/X/XScreen.kt` | Compose UI for X |
| Modify | `data/repository/XRepository.kt` | Add new method |
| ... | | |
 
## 5. Key Implementation Steps
Ordered list of what to build:
1. Data layer: model, DAO, or API call
2. Repository: method + interface
3. Use case / domain logic
4. ViewModel: state + events
5. UI: Compose screen + components
6. Navigation: wire up route
7. Tests: ViewModel unit tests + any critical logic tests
 
## 6. Edge Cases to Handle
- Empty state
- Error state (network, DB, validation)
- Loading state
- Offline behavior
- Configuration changes (rotation, process death)
 
## 7. Security Checklist for This Feature
- Any PII or sensitive data involved?
- Input validation needed?
- Auth/permission gating needed?
 
## 8. Test Plan (brief)
- What to unit test?
- What to manually test?
- Any UI test scenarios?
 
---
 
Once the plan is clear, proceed with implementation following the existing project conventions.
