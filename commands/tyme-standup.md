# /tyme-standup — Agile Standup (Tyme)

Generate a standup update for the Tyme team.

## Instructions

Look at git log, current branch, and Jira context to produce a concise agile standup.
```bash
git log --since="24 hours ago" --oneline
git status
git branch --show-current
```

Also fetch current Jira ticket if branch name contains ticket number:
```
getJiraIssue(ONC-XXX)  # extract ticket from branch name
```

---

## Standup Format

**Date**: [today]
**Sprint**: [current sprint if known]

### ✅ Yesterday
- ONC-XXX: [what was done] — [status: In Progress / PR raised / Merged]

### 🔨 Today
- ONC-XXX: [what will be worked on]

### 🚨 Blockers
- [ ] None
- [ ] Blocked on: [ticket / person / decision needed]

Keep it under 3 minutes to speak. One sentence per item.
