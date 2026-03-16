# /tyme-review — Code Review (Tyme Company)

Review the current code changes for: **$ARGUMENTS**

## Instructions

Self-review before raising a PR. Tyme is a fintech company — security and correctness are non-negotiable.
```bash
git diff main...HEAD --stat
git diff main...HEAD
```

---

## ✅ Jira Ticket Alignment
- [ ] Implementation matches the ticket acceptance criteria exactly
- [ ] Nothing extra added beyond the ticket scope (no scope creep)
- [ ] Any deviations from original approach noted in Jira comment

## 🔒 Fintech / Security (Tyme-specific)
- [ ] No financial data (amounts, account numbers) logged or exposed
- [ ] No PII in logs, analytics events, or error messages
- [ ] Payment flows have proper error handling — no silent failures
- [ ] Auth token handling follows Tyme's existing pattern exactly
- [ ] No new permissions added without team discussion
- [ ] Sensitive screens protected (screenshot prevention, biometric gate)

## 📐 Code Consistency (Team codebase)
- [ ] Follows the existing architecture — no new patterns introduced without discussion
- [ ] Naming matches existing conventions in the codebase
- [ ] No new dependencies added without team discussion
- [ ] No force unwrap (`!!` / `!`) — Tyme codebase uses safe unwrapping
- [ ] Localisation strings added for all user-facing text

## 📱 Platform Checklist
For Android:
- [ ] Follows existing MVVM/MVI pattern in the project
- [ ] No hardcoded strings — uses string resources
- [ ] Handles back navigation correctly
- [ ] Works on min SDK target

For iOS:
- [ ] Follows existing MVVM pattern in the project
- [ ] No hardcoded strings — uses Localizable.strings
- [ ] Handles navigation dismissal correctly
- [ ] Works on min iOS deployment target

## 🔁 CI Readiness
- [ ] Build passes locally: `./gradlew assembleDebug` or `Cmd+B`
- [ ] Unit tests pass: `./gradlew test` or `Cmd+U`
- [ ] No lint warnings introduced
- [ ] No TODO/FIXME comments left in changed files
- [ ] No debug/test code committed

## 📝 PR Readiness
- [ ] Branch is up to date with main/master
- [ ] Commit messages follow `ONC-XXX: type: description` format
- [ ] Ready to run `/tyme-pr ONC-XXX`

---

## Summary
**Issues Found** (🔴 Must Fix | 🟡 Should Fix | 🟢 Suggestion):
- ...

**Ready for PR**: Yes | No — fix issues first
