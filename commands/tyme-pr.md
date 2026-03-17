# /tyme-pr — Bitbucket PR + Jira Update

Create a Bitbucket PR description and update the Jira ticket for: **$ARGUMENTS**
Usage: `/tyme-pr ONC-123` or `/tyme-pr ONC-123 feat: add recurring payment`

## Instructions

This is the end-to-end handoff after implementation. Do all steps in order.

---

## Step 1 — Read the Jira Ticket

Use the Jira MCP to fetch the ticket:
- Get ticket title, description, acceptance criteria
- Note current status (should be "In Progress")
- Understand what was required
```
getJiraIssue(ONC-XXX)
```

---

## Step 2 — Read the Code Changes
```bash
git log main...HEAD --oneline
git diff main...HEAD --stat
git diff main...HEAD
```

Understand:
- What files changed and why
- What the implementation approach was
- Any notable decisions made

---

## Step 3 — Fetch Default Reviewers from Bitbucket

Use the Bitbucket MCP to get the repo's default reviewers.

First, detect workspace and repo slug from git remote:
```bash
git remote get-url origin
# e.g. git@bitbucket.org:tyme-bank/android-app.git
# workspace = tyme-bank, repo_slug = android-app
```

Then fetch default reviewers via Bitbucket MCP:
```
GET /2.0/repositories/{workspace}/{repo_slug}/default-reviewers
```

Extract the list of reviewer display names and account IDs.
- If reviewers found → include them in the PR reviewers list below
- If none configured → note "No default reviewers configured" and continue

---

## Step 4 — Generate Bitbucket PR Description

Produce a PR ready to paste into Bitbucket:

---

**Title**: `ONC-{ticketNumber}: <type>: <concise description>`
Example: `ONC-123: feat: add recurring payment screen`

**Reviewers** *(add these when creating the PR in Bitbucket)*:
- [Reviewer names from Step 3]

**Description**:

## Jira Ticket
[ONC-XXX](https://tyme.atlassian.net/browse/ONC-XXX) — [Ticket title]

## What
Concise description of what was built or changed.

## Why
What problem or requirement does this address? Reference the Jira acceptance criteria.

## How
Brief explanation of the technical approach. Highlight non-obvious decisions.

## Platform
- [ ] Android
- [ ] iOS
- [ ] Both

## Test Plan
How was this tested?
- [ ] Unit tests pass
- [ ] Build passes locally
- [ ] Manual: [specific scenario]
- [ ] Regression: [what existing features verified]

## Screenshots *(if UI changes)*
[Add screenshots]

## Checklist
- [ ] Code follows project conventions
- [ ] No debug logs or TODO without a linked ticket
- [ ] No force push, no commented-out code
- [ ] CI expected to pass

---

## Step 5 — Update Jira Ticket

Use the Jira MCP to add a comment to the ticket with implementation notes:
```
addCommentToJiraIssue(ONC-XXX, comment)
```

Comment format:
```
## Implementation Complete

**Branch**: feature/ONC-XXX-description
**PR**: [Bitbucket PR URL — add after creating PR]

**Approach**:
[2-3 sentences describing what was implemented and how]

**Notes for reviewer**:
[Anything the reviewer should know, edge cases, decisions made]

**Test coverage**:
[What was unit tested, what was manually tested]
```

---

## Step 6 — Checklist Before Submitting

- [ ] PR title follows format: `[ONC-XXXX] description`
- [ ] Default reviewers added to the PR (from Step 3)
- [ ] Jira ticket linked in PR description
- [ ] Jira ticket has implementation comment
- [ ] Branch is up to date with main/master (`git pull origin main`)
- [ ] Build passes locally
- [ ] No merge conflicts
- [ ] Self-review done (`/tyme-review`)
