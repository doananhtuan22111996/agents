# Mobile: Full ticket workflow (orchestrator)

$ARGUMENTS

This is the end-to-end workflow for implementing a mobile ticket. Each step maps to a focused skill that can also be run independently. Steps prefixed with `mobile-` are shared across platforms; steps prefixed with `ios-` or `android-` are platform-specific.

When the ticket affects both platforms, **Android and iOS implementation run as parallel sub-agents** — they are independent and do not block each other.

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
=== [Step 1] mobile-clarify ===
Status: PASSED
Requirement confirmed by user
---
=== [Step 2] Scope & Ticket IDs ===
Status: PASSED
KMP: yes (KMP-123) | Android: yes (AND-456) | iOS: yes (IOS-789)
---
=== [Step 3] kmp-implement ===
Status: PASSED
Files changed: 2 (TransferUseCase.kt, TransferRepository.kt)
Dev version: 1.5.0-dev-claude → published to mavenLocal
---
=== [Step 4] Platform implementation (parallel) ===
  [Android] android-implement: PASSED — 4 files changed, 8 tests added
  [iOS]     ios-implement:     PASSED — 3 files changed, 5 tests added
```

At the end of the full workflow, output a consolidated log:

```
=== TICKET SUMMARY: PNL-XXXXX ===
[Step 1]  requirement        : PASSED
[Step 2]  scope & tickets    : PASSED — KMP + Android + iOS
[Step 3]  kmp-implement      : PASSED | SKIPPED
[Step 4]  platform implement : PASSED (Android: PASSED, iOS: PASSED)
[Step 5]  platform precheck  : PASSED (Android: PASSED, iOS: PASSED)
[Step 6]  commit & PR        : PASSED — KMP PR #10, Android PR #11, iOS PR #12
[Step 7]  snapshot           : PASSED — snapshot branches pushed
[Step 8]  verify             : PASSED — Android main app OK, iOS main app OK
[Step 9]  summary            : PASSED
[Step 10] retrospective      : PASSED — updated CLAUDE.md, memory
[Step 11] jira update        : PASSED — https://ambcba.atlassian.net/browse/PNL-XXXXX
```

## Step 1: Shared requirement understanding → `/mobile-clarify`

Run `/mobile-clarify` to complete:
1. Requirement understanding (what, which flows, which repo, which country)
2. Figma design review (if UI ticket)
3. **Requirement Confirmation (1b) → WAIT for user approval**

Do NOT proceed until the user approves the requirement confirmation.

## Step 2: Determine scope and resolve ticket IDs

Use the platform detection from `/mobile-clarify` Step 1 (already confirmed by user in the Requirement Confirmation). If the platform was not determined there, detect it now using ALL available signals:

### Platform detection (check all, combine evidence)

1. **Title prefix**: `[Android]`, `[iOS]`, `[KMP]`, `[KMM]`
2. **Labels**: `Android`, `iOS`, `KMM`, `Mobile`
3. **Description keywords**: scan for platform-specific terms:
   - Android signals: `Android`, `Kotlin`, `Compose`, `Hilt`, `Gradle`, `build.gradle`, `.kt` file references, `tc-mx-android-*` repo names
   - iOS signals: `iOS`, `Swift`, `UIKit`, `SwiftUI`, `CocoaPods`, `Podfile`, `Podspec`, `.swift` file references, `tc-mx-ios-*` repo names
   - KMP signals: `KMP`, `KMM`, `shared module`, `commonMain`, `iosMain`, `tc-mx-kmp-*` repo names
   - Both platforms: `mobile`, `both platforms`, `Android and iOS`, `cross-platform`
4. **Comments**: read Jira comments for platform mentions, repo names, or clarifications from QA/PM
5. **Linked tickets**: check if parent/sub-tasks have platform prefixes that indicate scope

### Determine scope

From the combined signals:
1. **Does it need KMP changes?** (new/modified shared models, use cases, repositories, API definitions, domain logic)
2. **Which platforms are affected?**
   - Android only
   - iOS only
   - Both Android and iOS

Ask the user to confirm the scope if signals conflict or are absent:
> "This ticket appears to need: [KMP changes: yes/no], [Android: yes/no], [iOS: yes/no]. Is that correct?"

### Resolve ticket IDs per repo (MANDATORY)

Use `/jira` to read the ticket, then determine which ticket ID to use for **each repo** that will have changes. This ensures every repo commits on the correct branch tied to its platform ticket.

**a) Check the ticket title prefix, labels, description, and comments (as detected above):**
- Android signals → ticket is for **Android presentation repo**
- iOS signals → ticket is for **iOS repo**
- KMP/KMM signals → ticket is for **KMP repo**
- Mixed/both signals → ticket spans multiple repos

**b) Determine ticket assignment per repo:**

| Scenario | Android repo | iOS repo | KMP repo |
|----------|-------------|----------|----------|
| `[Android]` only | Use parent ticket ID | N/A | Check for KMP sub-task if KMP changes needed; create one if missing |
| `[iOS]` only | N/A | Use parent ticket ID | Check for KMP sub-task if KMP changes needed; create one if missing |
| `[KMP]` / `[KMM]` only | Check for Android sub-task if needed; create if missing | Check for iOS sub-task if needed; create if missing | Use parent ticket ID |
| Both `Android` and `iOS` (or no prefix) | Check for Android sub-task; create if missing | Check for iOS sub-task; create if missing | Check for KMP sub-task if needed; create if missing |

**c) When creating sub-tasks** via `/jira`:
- Summary: `[Android] <parent summary>`, `[iOS] <parent summary>`, or `[KMP] <parent summary>`
- Same assignee, priority, and sprint as parent
- Link as sub-task of the parent ticket

This gives you a ticket ID per repo: `ANDROID_TICKET_ID`, `IOS_TICKET_ID`, and/or `KMP_TICKET_ID`.

### Set up branches for each repo

For each repo that will have changes:
1. Determine branch prefix from Jira issue type: **Bug** → `bugfix/`, everything else → `feature/`
2. Check current branch: `git branch --show-current`
3. If the branch name does NOT match the repo's ticket ID, create the correct branch:
   ```bash
   git stash && git checkout master && git pull && git checkout -b <prefix>/<TICKET-ID>-short-description && git stash pop
   ```

**Pass each repo's ticket ID to the corresponding skill** so commits and PRs use the correct ticket reference.

## Step 3: KMP implementation (if needed) → `/kmp-implement`

If KMP changes are needed, run `/kmp-implement` **first** — before any platform-specific work. This ensures:
- Shared logic is implemented and tested
- Dev version is published to local Maven
- Platform repos can consume the updated KMP dependency

After KMP is done:
- Commit & push → PR (`/bitbucket create pr`) → snapshot (`/mobile-snapshot`)
- **Wait for KMP to complete before proceeding to Step 4.**

If no KMP changes needed → skip to Step 4.

## Step 4: Platform implementation (parallel sub-agents for dual-platform)

Based on the scope determined in Step 2:

### Case A: Both Android and iOS need changes

Launch **two sub-agents in parallel**:

1. **Android sub-agent** runs the full Android pipeline:
   - `/android-implement` — research, plan, implement code + tests, local verify
   - **Implementation Review Gate → WAIT for user approval**
   - `/android-precheck` — Spotless, build + unit tests, coverage
   - `/mobile-precheck` — Sonar quality gate (>= 80% on new code)
   - **Version bump** (MANDATORY — bump `build.gradle.kts` minor version)

2. **iOS sub-agent** runs the full iOS pipeline:
   - `/ios-implement` — research, plan, implement code + tests, local verify
   - **Implementation Review Gate → WAIT for user approval**
   - `/ios-precheck` — SwiftLint strict, SwiftGen, Cuckoo mocks, build + unit tests with coverage
   - `/mobile-precheck` — Sonar quality gate (>= 80% on new code)
   - **Version bump** (MANDATORY — bump podspec minor version)

Both agents work independently and report back when done. If a precheck fails, the agent fixes and re-runs.

### Case B: Only Android needs changes

Run the Android pipeline directly (no sub-agent needed):
`/android-implement` → review gate → `/android-precheck` + `/mobile-precheck` → version bump

### Case C: Only iOS needs changes

Run the iOS pipeline directly (no sub-agent needed):
`/ios-implement` → review gate → `/ios-precheck` + `/mobile-precheck` → version bump

**Note**: Each platform flow includes its own precheck. Do NOT run prechecks separately from this orchestrator.

## Step 5: Commit & PR → `/bitbucket create pr`

For each repo with changes:
- Commit with `[<TICKET-ID>] <message>` format (use the repo's specific ticket ID from Step 2)
- Push and create Bitbucket PR targeting `master`

## Step 6: Snapshot → `/mobile-snapshot`

For each repo with changes:
- Create `snapshot/<branch>` and push
- CI auto-publishes snapshot version
- Switch back to feature branch

## Step 7: Verify on main app

Based on platforms:
- **iOS** → `/ios-verify` — update main app Podfile to local path, build, fix cascading deps, generate test cases
- **Android** → `/android-verify` — update main app dependency to dev/local version, build, fix cascading deps, generate test cases

If both platforms: run both verifications (can be parallel if different main app repos).

If build fails due to cascading dependencies: fix each broken repo (implement → precheck → PR → snapshot), update main app, repeat until build succeeds.

## Step 8: Summary

Report:
- Files changed and why (per repo)
- Tests added (per repo)
- Sub-module PR link(s)
- Snapshot version(s) published
- Cascading repo PRs (if any)
- Main app verification result (per platform)
- Manual test cases

## Step 9: Retrospective → `/mobile-retrospective`
- Update CLAUDE.md with new repo-specific knowledge
- Update skills if any had gaps or friction
- Update auto-memory with cross-session learnings

## Step 10: Jira ticket update → `/jira`

Update the Jira ticket with the **final implementation details**.

**Decision: Update description vs. Add comment**

First, check if the ticket already has a meaningful description (e.g., written by QA with preconditions, steps to reproduce, expected/actual results, screenshots, or videos).

- **If the ticket has an existing description from QA/PM**: Do NOT overwrite it. Instead, **add a comment** with the implementation details. The original description is valuable for QA and future reference.
- **If the ticket description is empty or only contains the raw input from the user**: **Replace the description** with the structured implementation details.

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
| Repo | Files | Description |
|------|-------|-------------|
| KMP  | <list> | <changes> |
| Android | <list> | <changes> |
| iOS | <list> | <changes> |

### PRs & Repos
- KMP: <repo name> — PR <link> (version <X.Y.Z>)
- Android: <repo name> — PR <link> (version <X.Y.Z>)
- iOS: <repo name> — PR <link> (podspec <X.Y.Z>)
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
| Repo | Pattern | New files | Modified files |
|------|---------|-----------|---------------|
| KMP | <Clean Architecture> | <list> | <list> |
| Android | <MVVM + Compose> | <list> | <list> |
| iOS | <MVVM-C + RxSwift> | <list> | <list> |

## Scope
| Area | KMP | Android | iOS |
|------|-----|---------|-----|
| Domain/API | Yes/No | — | — |
| UI | — | Yes/No | Yes/No |
| Tests | Yes/No | Yes/No | Yes/No |
| Version bump | Yes/No | Yes/No | Yes/No |

## Technical Notes
- <Gotchas, workarounds, things a reviewer should know>

## Affected Screens
- <Which screens/flows are impacted>

## Repos & PRs
- KMP: <repo name> — PR <link>
- Android: <repo name> — PR <link>
- iOS: <repo name> — PR <link>
- Country config: <GTZA/SLZA/GTPH>
```

**How to push to Jira:**
1. Fetch the ticket description first: GET `{JIRA_BASE_URL}/rest/api/3/issue/{TICKET_KEY}?fields=description`
2. Check if description has meaningful content (more than just a title or empty)
3. Read Jira credentials from `~/.claude.json` → `mcpServers.jira.env`
4. If existing description is meaningful → POST comment
5. If description is empty/minimal → PUT description
6. Confirm with link: `https://ambcba.atlassian.net/browse/{TICKET_KEY}`

---

Execute each step in order. **MANDATORY pause points** where you MUST stop and wait for user approval:
1. **Step 1** — Requirement Confirmation (before any implementation)
2. **Step 4** — Implementation Review Gate per platform (before precheck)

Between all other steps, proceed without pausing. If a step fails, fix the issue and re-run that step, then continue to the next. Only stop the entire workflow if a step fails and you cannot fix it. Each skill can be re-run independently if needed.

## Multi-repo ordering (CRITICAL)

When a ticket spans multiple repos, follow this order strictly:
1. **KMP repo first** (if needed): implement → precheck → commit & push → PR → snapshot
2. Wait for KMP to be ready (user confirms)
3. **Platform repos** (Android and/or iOS): can run **in parallel** since they both depend on KMP, not on each other
4. Each platform: implement → precheck → commit & push → PR → snapshot → verify on main app

**NEVER** start platform implementation before KMP changes are complete and published.

## Mid-ticket changes

When the user requests changes after initial implementation (e.g., code review feedback):
- Treat it as a **new implementation cycle**: re-implement → re-run full precheck → commit → push → update snapshot
- Do NOT skip precheck steps just because the change seems small
- Any code change can break tests or introduce lint violations — always verify

## Input template

The `$ARGUMENTS` should include enough detail to implement without guesswork. If missing key info, ask the user before proceeding.

```
Ticket: PNL-XXXXX
Repo(s): <list of repos involved>
Country: GTZA / SLZA / GTPH / keep current
Platforms: Android / iOS / Both

## What to implement
<Detailed description of the change:>
- What to add/remove/modify
- New models, use cases, API changes
- UI changes (screens, components, navigation)

## Scope
- KMP change? yes/no
- Android change? yes/no
- iOS change? yes/no
- Breaking change? yes/no — if yes, which downstream repos need adaptation

## Downstream repos that need updating
<List repos that depend on changed APIs, and what they need to change>

## Main app location
- Android: <local path to Android main app repo>
- iOS: <local path to iOS main app repo>

## Acceptance criteria
<Specific, testable conditions that define "done">
```

$ARGUMENTS
