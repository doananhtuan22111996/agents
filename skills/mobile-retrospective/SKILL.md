# Mobile: Retrospective — update knowledge after completing a ticket

Review what happened during the ticket and update knowledge for future tickets.

## 1. Update CLAUDE.md (repo-specific knowledge)

Read the current `CLAUDE.md` in the repo root. Add or update if any of these are new:
- **Architecture patterns** discovered during this ticket
- **Key files/protocols** that were non-obvious to find or important to understand
- **Business rules** that affected implementation
- **Build/test gotchas** specific to this repo
- **Model field names** that were confusing or misused

Do NOT add generic rules that apply to all repos — those go in skills or memory.

## 2. Update skills (generic improvements)

Review if any skill had a gap or friction during this ticket:
- Did a step miss something important?
- Did a check fail in a way the skill didn't handle?
- Was there a manual workaround that should be automated?

If yes, update the skill file in `.claude/commands/` so the next ticket benefits.

Skills to review:
- Shared: `/mobile-implement`, `/mobile-precheck`, `/mobile-snapshot`, `/mobile-retrospective`, `/bitbucket`, `/figma`
- iOS: `/ios-implement`, `/ios-precheck`, `/ios-verify`, `/ios-ticket`
- Android: `/android-implement`, `/android-precheck`, `/android-verify`, `/android-ticket`

## 3. Update auto-memory (cross-session learnings)

Save to memory if:
- A **pitfall** was hit that could recur
- A **debugging technique** was useful
- A **cross-repo pattern** was discovered

## 4. Report updates

List what was updated and why, so the user can review:
- CLAUDE.md changes
- Skill file changes
- Memory file changes

If nothing needs updating, confirm: "No updates needed — existing docs covered this ticket well."

## Safety

- Never write tokens, keys, or credentials to CLAUDE.md, skill files, or memory files
- Never write sensitive repo paths or internal URLs that shouldn't be shared

## 5. Log output

```
=== [mobile-retrospective] ===
Status: PASSED
CLAUDE.md: updated | no changes
Skills updated: <list, or "none">
Memory updated: <list, or "none">
```
