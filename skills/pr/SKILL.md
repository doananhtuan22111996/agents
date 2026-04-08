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

---

## Create PR

**Rule: Always use MCP GitHub tools to create PRs. Never use `gh` CLI.**

Once the PR title and body are generated above, create the pull request:

1. Ensure all changes are committed and pushed to the remote branch (use `git push`)
2. Use the MCP GitHub `create_pull_request` tool to create the PR — do NOT use `gh` CLI
3. If the base branch is not `main`, specify it in the MCP tool call
4. Return the PR URL to the user
