# Mobile: Requirement clarification and review gates

$ARGUMENTS

This skill contains the **platform-agnostic** requirement clarification and review gate steps. It is called by `/mobile-ticket` and platform-specific skills (`/ios-implement`, `/android-implement`).

## Step 1: Understand the requirement

Read the ticket description and clarify:
1. **What** needs to change (new feature, bug fix, refactor)?
2. **Which flows/screens** are affected?
3. **Which repo** is the primary target?
4. **Which country/target?** Check the ticket for country context:
   - **GTZA** (GoZA / GoSA) — South Africa TymeBank
   - **SLZA** (Sanlam) — South Africa Sanlam
   - **GTPH** (GoPH) — Philippines
   - If no country is mentioned, keep the current default target

5. **Which platform(s)?** Detect affected platforms using ALL available signals (check in order, combine evidence):

   **a) Title prefix**: `[Android]`, `[iOS]`, `[KMP]`, `[KMM]`
   **b) Labels**: `Android`, `iOS`, `KMM`, `Mobile`
   **c) Description keywords**: scan for platform-specific terms:
   - Android signals: `Android`, `Kotlin`, `Compose`, `Hilt`, `ViewModel`, `Gradle`, `build.gradle`, `.kt` file references, `tc-mx-android-*` repo names
   - iOS signals: `iOS`, `Swift`, `UIKit`, `SwiftUI`, `CocoaPods`, `Podfile`, `Podspec`, `.swift` file references, `tc-mx-ios-*` repo names
   - KMP signals: `KMP`, `KMM`, `shared module`, `commonMain`, `iosMain`, `tc-mx-kmp-*` repo names
   - Both platforms: `mobile`, `both platforms`, `Android and iOS`, `cross-platform`
   **d) Comments**: read Jira comments for platform mentions, repo names, or clarifications from QA/PM about which platforms are affected
   **e) Linked tickets**: check if parent/sub-tasks have platform prefixes that indicate scope

   If signals conflict or are absent, ask the user to confirm.

If anything is unclear or ambiguous, **ask the user before proceeding**. Common questions:
- Does this apply to all flows or specific ones?
- Are there UI changes or just logic?
- Does this affect a public API module or just the implementation?
- Which country/target should we build for?
- Which platform(s) are affected? (Android / iOS / Both)

Do NOT start coding until the requirement is clear.

### Step 1a: Read linked external content (MANDATORY — before anything else)

After reading the Jira ticket, **scan the description and comments for external links** (Slack threads, Confluence pages, Figma designs). These often contain critical context not in the ticket itself. Read them **before** forming your understanding.

**Slack thread links** (e.g., `https://tymex.slack.com/archives/<channel_id>/p<timestamp>`):
- Parse the URL to extract `channel_id` and `thread_ts` (insert `.` so 6 digits follow: `p1772422863758569` → `1772422863.758569`)
- Use `mcp__slack__slack_get_thread_replies` with `channel_id` and `thread_ts` to read the full thread
- If the Slack MCP read tool is not available, ask the user:
  > "The ticket links to a Slack thread: `<URL>`. I can't read Slack threads directly. Could you paste the relevant messages or summarize the context?"
- Extract: the reported problem, reproduction steps, expected vs actual behavior, any decisions or conclusions reached in the thread

**Confluence page links** (e.g., `https://ambcba.atlassian.net/wiki/spaces/<SPACE>/pages/<PAGE_ID>/...`):
- Extract the page ID from the URL (numeric segment after `/pages/`)
- Use `mcp__confluence__conf_get` to read the page:
  ```
  path: "/wiki/api/v2/pages/<PAGE_ID>/body"
  queryParams: {"body-format": "atlas_doc_format"}
  ```
- Extract: requirements, acceptance criteria, technical specs, test data, or any other implementation-relevant content

**Figma design links** (if UI ticket):
- Use `/figma read <URL>` to extract screen layouts, component hierarchy, text content, and states
- Cross-reference the Figma design with the requirement to ensure alignment
- Note any discrepancies between the design and the ticket description

**Also check Jira comments** — they often contain clarifications, decisions, or updated requirements:
- Use `mcp__jira__jira_get` with path `/rest/api/3/issue/<TICKET_KEY>/comment` to read all comments
- Look for additional links (Slack, Confluence, Figma) in the comments too

Combine all gathered context (ticket description + external links + comments) before proceeding to requirement clarification.

### Step 1b: Requirement Confirmation (MANDATORY — do NOT skip)

After reading the ticket and researching the codebase, you MUST present a **Requirement Confirmation** to the user and **wait for approval** before writing any code. This prevents wasted effort from misunderstood requirements.

**Format:**

```
=== Requirement Confirmation ===

Ticket: <TICKET_KEY>
Platform(s): <Android / iOS / Both / KMP + Android / KMP + iOS / KMP + Both>
Country: <GTZA / SLZA / GTPH / unchanged>
My understanding: <1-2 sentences of what the ticket is asking>

Expected behavior: <what should happen after the fix>
Current behavior: <what currently happens (the bug/gap)>

Proposed approach:
- <Bullet points describing what you plan to change and why>
- <Which files/areas will be modified>
- <Which repo(s) will be touched>

Platform detection signals:
- <List what signals led to the platform determination — e.g., "title prefix [Android]", "description mentions tc-mx-ios-transfer", "comment from QA says both platforms">

Questions (if any):
- <Any remaining ambiguities>

Please confirm this understanding is correct before I proceed.
```

**Rules:**
- ALWAYS present this confirmation and WAIT for user response
- If the user corrects your understanding, update your approach before proceeding
- For bug fix tickets: pay close attention to which value is "expected" vs "actual" — the ticket title/description format is: "Show incorrect X instead of Y" means Y is the CORRECT value
- For string/wording tickets: identify ALL locations where the string appears (all localization files, all countries) and list them in the confirmation
- Do NOT assume the fix approach — verify by reading the actual code and strings first

## Implementation Review Gate (MANDATORY — do NOT skip)

After completing the implementation (code changes + tests), you MUST:
1. Present a summary of all files changed, what was changed and why
2. Ask the user to review and approve
3. **STOP and WAIT** for explicit user approval (e.g., "ok", "approved", "go ahead", "lgtm")
4. Only proceed to the next step (precheck) after receiving approval
5. If the user requests changes, apply them and present the updated summary again

**Do NOT auto-proceed to precheck.** The user must explicitly approve the implementation first.
