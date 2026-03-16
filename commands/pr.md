# /pr — Pull Request Description
 
Generate a pull request title and body for: **$ARGUMENTS**
 
## Instructions
 
Look at the current git diff and recent commits, then produce a PR ready to submit.
 
First, run:
```
git diff main...HEAD --stat
git log main...HEAD --oneline
```
 
Then produce:
 
---
 
## PR Title
Format: `<type>(<scope>): <concise description>`
Example: `feat(tasks): add recurring task support`
 
## PR Body
 
### What
Concise description of what changed. What is the new behavior?
 
### Why
What problem does this solve? Why is this change needed now?
 
### How
Brief explanation of the technical approach. Highlight any non-obvious decisions.
 
### Screenshots / Demo *(if UI changes)*
[Add screenshots here]
 
### Test Plan
How was this tested? What should the reviewer verify?
- [ ] Unit tests pass: `./gradlew test`
- [ ] App builds: `./gradlew assembleDebug`
- [ ] Manual: [specific scenario tested]
- [ ] Regression: [what existing features were verified]
 
### Checklist
- [ ] Code follows project conventions
- [ ] No debug code or TODO comments left without a task
- [ ] No sensitive data exposed
- [ ] Tests added for new logic
- [ ] CLAUDE.md updated if conventions changed
 
---
 
**Labels**: (suggest based on type: `feature`, `bug`, `chore`, `refactor`, `docs`)
**Branch**: `<type>/<short-description>`
