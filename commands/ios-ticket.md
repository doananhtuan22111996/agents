# iOS: Full ticket workflow (orchestrator)

$ARGUMENTS

This is the end-to-end workflow for implementing an iOS ticket. Each step maps to a focused skill that can also be run independently. Steps prefixed with `mobile-` are shared across platforms; steps prefixed with `ios-` are iOS-specific.

## Safety guardrails

These rules apply to ALL steps, especially in `--dangerously-skip-permissions` mode:

**NEVER do:**
- Print, log, or echo any token, key, secret, or credential (e.g., `SONAR_TOKEN`, Bitbucket tokens, API keys)
- Pass tokens in command output or tool result text — use env vars (`$SONAR_TOKEN`) only
- Push to any repo other than the ones listed in the ticket (sub-module, downstream, main app)
- Publish, upload, or send data to external services other than: Bitbucket (PR/push), SonarCloud (scan)
- Delete remote branches on `master` or `main`
- Force push to any shared branch
- Modify `~/.claude/settings.json` or any credential files
- Read or display contents of `.env`, credentials, keychain, or token files

**ALWAYS do:**
- Use `$SONAR_TOKEN` env var reference — never hardcode or print the value
- Only `git push` to branches that match `feature/*` or `snapshot/*` patterns
- Only create PRs targeting `master` on repos explicitly in the ticket scope
- Confirm with user before pushing to any repo not mentioned in the ticket

## Logging

Each step MUST output a clear log header and result so issues can be traced:

```
=== [Step N] <Skill Name> ===
Status: STARTED | PASSED | FAILED | SKIPPED
Details: <what happened>
Duration: <if relevant>
```

Example:
```
=== [Step 1] mobile-implement + ios-implement ===
Status: PASSED
Files changed: 3 (AccountsSelectionProvider.swift, +UseCase.swift, +Mapping.swift)
Tests added: 5
---
=== [Step 2] ios-precheck + mobile-precheck ===
Status: FAILED
SwiftLint: PASSED (0 violations)
Build + Tests: FAILED — 2 test failures in AccountsSelectionProviderImplTests
Details: testStartAccountSelection assertion failed at line 142
```

At the end of the full workflow, output a consolidated log:

```
=== TICKET SUMMARY: PNL-XXXXX ===
[Step 1] implement         : PASSED
[Step 2] precheck          : PASSED (retry 1)
[Step 3] bitbucket PR      : PASSED — PR #10
[Step 4] snapshot          : PASSED — snapshot/PNL-XXXXX
[Step 5] verify            : PASSED — tb-gosa-ios-app builds OK
[Step 6] Summary           : PASSED
[Step 7] retrospective     : PASSED — updated CLAUDE.md, memory
[Step 8] jira update       : PASSED — https://ambcba.atlassian.net/browse/PNL-XXXXX
```

## Step 1: Implement → `/mobile-implement` + `/ios-implement`

1. `/mobile-implement` — Understand requirement, Figma review (if UI), **Requirement Confirmation (1b) → WAIT for user approval**
2. `/ios-implement` — Research, plan, implement code + tests, local verify
3. **Implementation Review Gate → WAIT for user approval** (defined in `/mobile-implement`)

## Step 2: Pre-PR checks → `/ios-precheck` + `/mobile-precheck`
- `/ios-precheck` — SwiftLint strict, SwiftGen, Cuckoo mocks, build + unit tests with coverage
- `/mobile-precheck` — Sonar quality gate (>= 80% on new code) — **do NOT skip Sonar**. `$SONAR_TOKEN` is in `~/.claude.json` global env.
- If checks fail, fix and re-run
- **MUST bump podspec versions** before this step (every ticket needs a version bump)

## Step 3: Commit & PR → `/bitbucket create pr`
- Commit with `[PNL-{ID}] {message}` format
- Push and create Bitbucket PR targeting `master`

## Step 4: Snapshot → `/mobile-snapshot`
- Create `snapshot/<branch>` and push
- CI auto-publishes snapshot version
- Switch back to feature branch

## Step 5: Verify on main app → `/ios-verify`
- Update main app Podfile to local path for changed sub-module
- Build main app
- If build fails due to cascading dependencies: fix each broken repo (implement → precheck → PR → snapshot), update main app, repeat until build succeeds
- If build succeeds: generate manual test cases for self-verification

## Step 6: Summary
Report:
- Files changed and why
- Tests added
- Sub-module PR link(s)
- Snapshot version(s) published
- Cascading repo PRs (if any)
- Main app verification result
- Manual test cases

## Step 7: Retrospective → `/mobile-retrospective`
- Update CLAUDE.md with new repo-specific knowledge
- Update skills if any had gaps or friction
- Update auto-memory with cross-session learnings

## Step 8: Jira ticket update → `/jira`
Update the Jira ticket with the **final implementation details**.

**Decision: Update description vs. Add comment**

First, check if the ticket already has a meaningful description (e.g., written by QA with preconditions, steps to reproduce, expected/actual results, screenshots, or videos).

- **If the ticket has an existing description from QA/PM**: Do NOT overwrite it. Instead, **add a comment** with the implementation details. The original description is valuable for QA and future reference.
- **If the ticket description is empty or only contains the raw input from the user** (e.g., just a one-liner or the arguments passed to this skill): **Replace the description** with the structured implementation details.

### Comment format (when preserving existing description)

Build the ADF JSON and POST to `{JIRA_BASE_URL}/rest/api/3/issue/{TICKET_KEY}/comment` with `{"body": <ADF>}`.

**Comment content:**

```
## Implementation Details

### Root Cause / What Changed
<1-2 sentences explaining the root cause or what was implemented>

### Fix / Implementation
- <Bullet points describing the changes>

### Files Changed
- <List of files with brief description of each change>

### PR & Repo
- Repo: <repo name>
- PR: <link to PR>
- Podspec: <version>
```

### Description format (when replacing empty/minimal description)

Build the ADF JSON and PUT to `{JIRA_BASE_URL}/rest/api/3/issue/{TICKET_KEY}` with `{"fields":{"description": <ADF>}}`.

**Description content** (convert to Atlassian Document Format / ADF):

```
## Summary
<1-2 sentences: what was implemented and why>

## Requirements
### Display Conditions / Business Rules
- <Bullet points describing when/how the feature activates>

### UI Implementation
- <Components used, navigation behavior, user interactions>

### Configuration
- <Feature toggles, remote config keys, JSON formats, local defaults>

## Architecture
- **Pattern**: <MVVM-C, RxSwift, etc.>
- **New files**: <list with brief description of each>
- **Modified files**: <list with what changed>

## Scope
| Area | Changed? |
|------|----------|
| Api pod | Yes/No |
| UI | Yes/No — description |
| Tests | Yes/No — count |
| Podspec version | Yes/No |

## Technical Notes
- <Gotchas, workarounds, things a reviewer should know>

## Affected Screens
- <Which screens/flows are impacted>

## Repo & Environment
- **Repo**: <repo name>
- **Main app**: <path>
- **Country config**: <GTZA/SLZA/GTPH>
```

**How to push to Jira:**
1. Fetch the ticket description first: GET `{JIRA_BASE_URL}/rest/api/3/issue/{TICKET_KEY}?fields=description`
2. Check if description has meaningful content (more than just a title or empty)
3. Read Jira credentials from `~/.claude.json` → `mcpServers.jira.env`
4. If existing description is meaningful → POST comment to `{JIRA_BASE_URL}/rest/api/3/issue/{TICKET_KEY}/comment` with `{"body": <ADF>}`
5. If description is empty/minimal → PUT to `{JIRA_BASE_URL}/rest/api/3/issue/{TICKET_KEY}` with `{"fields":{"description": <ADF>}}`
6. Confirm with link: `https://ambcba.atlassian.net/browse/{TICKET_KEY}`

---

Execute each step in order. **MANDATORY pause points** where you MUST stop and wait for user approval:
1. **Step 1b** — Requirement Confirmation (before writing code)
2. **Implementation Review Gate** — after Step 1 completes (before precheck)

Between all other steps, proceed without pausing. If a step fails, fix the issue and re-run that step, then continue to the next. Only stop the entire workflow if a step fails and you cannot fix it. Each skill can be re-run independently if needed.

## Multi-repo ordering (CRITICAL)

When a ticket spans multiple repos, **complete each repo fully before moving to the next**:
1. Repo A: implement → precheck → commit & push → PR → snapshot
2. Wait for snapshot to publish (user confirms)
3. Repo B: update Podfile.swift with Repo A's snapshot version → implement → precheck → commit & push → PR → snapshot
4. Repeat for additional repos

**NEVER** implement across multiple repos simultaneously — it causes version mismatch issues.

## Mid-ticket changes

When the user requests changes after initial implementation (e.g., code review feedback):
- Treat it as a **new implementation cycle**: re-implement → re-run full precheck (build, tests, SwiftLint) → commit → push → update snapshot
- Do NOT skip precheck steps just because the change seems small
- Any code change can break tests or introduce lint violations — always verify

## Input template

The `$ARGUMENTS` should include enough detail to implement without guesswork. If missing key info, ask the user before proceeding.

```
Ticket: PNL-XXXXX
Repo: <sub-module repo name>
Country: GTZA / SLZA / GTPH / keep current

## What to implement
<Detailed description of the change:>
- What to add/remove/modify
- New protocol methods and their signatures/behavior
- Callback parameters and expected values
- What existing methods to remove or rename

## Affected flows/screens
<Which screens, modules, protocols are impacted>

## Scope
- API pod change? yes/no
- UI change? yes/no
- Affects single account flow? yes/no
- Affects multiple account flow? yes/no
- Breaking change? yes/no — if yes, which downstream repos need adaptation

## Downstream repos that need updating
<List repos that depend on changed APIs, and what they need to change>

## Main app location
<Local path to main app repo for verification>

## Acceptance criteria
<Specific, testable conditions that define "done">
```
