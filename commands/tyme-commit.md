# /tyme-commit — Commit for Tyme Workflow

Generate a commit message and push for: **$ARGUMENTS**
Usage: `/tyme-commit ONC-123` or `/tyme-commit ONC-123 feat: add payment screen`

## Instructions

Look at current staged changes and produce the right commit message for Tyme conventions.

---

## Step 1 — Check Staged Changes
```bash
git status
git diff --cached --stat
git diff --cached
```

If nothing staged, remind: `git add <files>` first.

---

## Step 2 — Generate Commit Message

**Format**:
```
ONC-{ticketNumber}: <type>(<scope>): <description>

[optional body — if change is complex]
[explain WHY, not what — the diff shows what]
```

**Types**: `feat` | `fix` | `refactor` | `test` | `chore` | `docs` | `perf`

**Examples**:
```
ONC-123: feat(payment): add recurring payment screen

ONC-456: fix(auth): resolve token refresh loop on expired session

ONC-789: refactor(tasks): extract TaskMapper to separate class
```

**Rules**:
- Ticket number always first
- Description lowercase, no period at end
- Max 72 chars on first line
- Body explains WHY if non-obvious
- Never include `!!`, debug logs, or TODOs in committed code

---

## Step 3 — Commit Command
```bash
git commit -m "ONC-XXX: type(scope): description"
```

---

## Step 4 — Push to Bitbucket
```bash
# First push — set upstream
git push -u origin feature/ONC-XXX-description

# Subsequent pushes
git push
```

---

## Step 5 — Next

After pushing:
→ Use `/tyme-pr ONC-XXX` to create the PR description and update Jira
