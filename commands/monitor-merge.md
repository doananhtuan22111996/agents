---
description: Monitor an open Bitbucket PR until enough approvals, then merge
---

# Monitor my open PRs and auto-merge when approved

Automatically find my most recent open PR on Bitbucket and monitor it until it gets 2 approvals, then merge.

## Arguments

- Optional: `<pr_number>` or `<pr_number> <required_approvals>`. If omitted, auto-detect the most recent open PR by the current user.

## Instructions

1. **Determine which PR to monitor:**
   - If a PR number is provided in arguments, use that directly.
   - If **no** arguments provided:
     - Use `mcp__bitbucket__getPullRequests` with workspace `tymerepos`, repo `tc-mx-mobile-e2e-automation-testing-svc`, and state `OPEN` to list open PRs.
     - Filter to find PRs authored by the current user (match display name containing "Thong Nguyen" or nickname containing "Thong").
     - Pick the most recently created one.
     - If no open PR found, report that and stop.
   - Default required approvals: **2** (override via second argument).

2. **Immediate check** using `mcp__bitbucket__getPullRequest`:
   - workspace: `tymerepos`
   - repo_slug: `tc-mx-mobile-e2e-automation-testing-svc`
   - pull_request_id: the PR number

3. Count participants where `approved: true`.

4. **If enough approvals:**
   - Use `mcp__bitbucket__mergePullRequest` to merge (`merge_strategy`: `merge_commit`, `close_source_branch`: `true`).
   - Report success: PR number, title, who approved, merged status.

5. **If not enough approvals yet:**
   - Report current status: PR number, title, current approvals (list who approved), how many more needed.
   - Set up a recurring check every **15 minutes** using `/loop 15m` with a prompt to:
     - Check the PR using `mcp__bitbucket__getPullRequest` (workspace: `tymerepos`, repo: `tc-mx-mobile-e2e-automation-testing-svc`).
     - Count approved participants.
     - If enough approvals: merge with `mcp__bitbucket__mergePullRequest` (`merge_strategy`: `merge_commit`, `close_source_branch`: `true`), report success, and cancel the cron job using CronDelete.
     - If not enough: report current approval count and continue waiting.
   - Tell the user the cron job ID so they can cancel with CronDelete if needed.

## Fallback if MCP merge fails (HTTP 400)

If `mergePullRequest` returns an error, merge via Bitbucket REST API using Keychain-backed credentials:

`~/.cursor/scripts/bitbucket-merge-pr.sh tymerepos tc-mx-mobile-e2e-automation-testing-svc <pull_request_id>`

(Ensure `~/.cursor/scripts/setup-bitbucket-keychain.sh` has been run once.)
