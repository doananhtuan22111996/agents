# /review — Self Code Review
 
Review the current code changes (or: $ARGUMENTS) against the full checklist.
 
## Instructions
 
Perform a thorough self-review. Be critical. The goal is to catch issues before a PR is opened. Look at the actual diff or the files involved.
 
---
 
## Review Checklist
 
### ✅ Correctness
- [ ] Logic is correct for all inputs and scenarios
- [ ] All edge cases handled: empty, null, boundary values, concurrent access
- [ ] No off-by-one errors, wrong comparisons, or logic inversions
- [ ] State updates are atomic where needed
- [ ] Suspend functions / Flows are called from correct coroutine scope
 
### 🔒 Security
- [ ] No sensitive data (tokens, PII, passwords) in logs, errors, or analytics
- [ ] Auth checks in place before accessing protected data/screens
- [ ] Input validated and sanitized before use
- [ ] No hardcoded secrets or API keys
- [ ] Sensitive data uses EncryptedSharedPreferences or equivalent
- [ ] Deep link / intent data is validated
 
### ⚡ Performance
- [ ] No unnecessary recompositions (Compose `remember`, `derivedStateOf` used correctly)
- [ ] No N+1 database queries
- [ ] Heavy work off the main thread (IO dispatcher used)
- [ ] No memory leaks: coroutine scopes tied to lifecycle, no static context refs
- [ ] Bitmaps / large data not held longer than needed
 
### 🛡️ Error Handling
- [ ] Network errors handled gracefully (not a crash)
- [ ] DB errors handled gracefully
- [ ] Unknown/unexpected errors caught and logged (not silently swallowed)
- [ ] User sees meaningful error messages (not raw stack traces)
- [ ] Retry logic where appropriate
 
### 🧪 Testability & Tests
- [ ] Critical business logic is unit tested
- [ ] ViewModel state changes tested
- [ ] Tests are readable and test behavior, not implementation
- [ ] No untested `!!` force-unwraps in production paths
 
### 🧹 Code Quality
- [ ] Names are clear and self-documenting (no `data2`, `temp`, `stuff`)
- [ ] Functions are focused and small (single responsibility)
- [ ] No dead code or commented-out code left behind
- [ ] No TODO comments without a linked task
- [ ] Follows existing project conventions (naming, structure, patterns)
 
### 🏗️ Architecture
- [ ] Correct layer owns this logic (no business logic in Composables)
- [ ] No Android dependencies in domain/use-case layer
- [ ] Repository abstracts data sources correctly
- [ ] ViewModel doesn't hold Context references
- [ ] Navigation handled at correct level
 
### 📱 Android Specifics
- [ ] Handles configuration changes correctly (rotation, dark mode, font size)
- [ ] Works in low-memory / process death scenarios
- [ ] Permissions requested correctly and handled when denied
- [ ] Works offline (or degrades gracefully)
- [ ] Respects system back gesture
 
---
 
## Summary
After reviewing, produce:
 
**Issues Found** (categorized as: 🔴 Must Fix | 🟡 Should Fix | 🟢 Nice to Have):
- ...
 
**Overall Assessment**: Ready to merge | Needs changes | Significant rework needed
