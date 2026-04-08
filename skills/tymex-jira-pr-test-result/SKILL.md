---
name: tymex-jira-pr-test-result
description: >-
  Posts TymeX-style explicit test results to Jira (table, env, device, #testresult),
  updates Bitbucket PR description with focusedCommentId link, syncs Jira description
  and edits stale comments when PRs are replaced. Use when preparing a PR for merge,
  branch renames, TymeX testing guidelines, Jira test table, CB-1256 format, or
  linking PR description to a Jira test comment.
---

# TymeX: Jira test result + PR link

## When to apply

- User (or workflow) must document **self-test / test results on the Jira ticket** before merge, per TymeX practice.
- User provides a **Bitbucket PR URL** (or workspace, repo slug, PR id) and a **Jira issue key** (or infer from PR title/branch, e.g. `PNL-12345`).

## Canonical references (do not invent policy)

- **Guideline:** [TymeX — Engineers need to explicit test result before merge pull request](https://ambcba.atlassian.net/wiki/spaces/TM/pages/3737190876/TymeX+Engineers+need+to+explicit+test+result+before+merge+pull+request) — document results on the ticket; reviewers check the ticket; if no testing applies, comment `Test result: None`.
- **Format example:** [CB-1256](https://ambcba.atlassian.net/browse/CB-1256) — “Test result”, bullet **Env** / **Device**, markdown table **Cases | Result**, hashtag `#testresult`.

## Tools to use

Prefer MCP when available:

| Action | Atlassian MCP | Bitbucket MCP |
|--------|---------------|---------------|
| Read guideline page | `confluence_get_page` with `page_id` from URL | — |
| Read ticket / comments | `jira_get_issue` | — |
| Post test comment | `jira_add_comment` (body: Markdown) | — |
| Fix test comment (wrong PR URL) | `jira_edit_comment` (`issue_key`, `comment_id`, `body`) | — |
| Update ticket description | `jira_update_issue` (`fields` JSON; Jira wiki/markdown per instance) | — |
| PR title & description | — | `getPullRequest` |
| Prepend PR description with Jira link | — | `updatePullRequest` (`description` only; omit `title` to keep title) |
| Replace PR (branch rename, new PR id) | — | `declinePullRequest`, then `createPullRequest` — **`updatePullRequest` cannot change source branch** |

**Focused Jira comment URL** (for PR description and reviewers):

`https://ambcba.atlassian.net/browse/<KEY>?focusedCommentId=<commentId>`

Read `<commentId>` from the `jira_add_comment` response (`id` field).

## Workflow

1. **Resolve identifiers**
   - Jira key: from user, from PR title `[PNL-12345] …`, or branch `feature/PNL-12345-…`.
   - Prefer **branch names that include the ticket key** (`feature/PNL-12345-short-slug`) **before** the first PR — avoids superseding PRs and stale Jira links later.
   - Bitbucket: parse `https://bitbucket.org/<workspace>/<repo>/pull-requests/<id>` → `getPullRequest` with `workspace`, `repo_slug`, `pull_request_id`.

2. **Draft test cases** from PR title, PR description bullets, and (if repo is open locally) the diff or touched files. Rows should be **specific, observable** checks (navigation, UI, regression, unit tests). Align **Env** / **Device** with what was actually run; if unknown, use a placeholder and tell the user to edit.

3. **Post the Jira comment** using the template below. Include **PR title** and **PR URL** at the top when the user wants traceability to a specific PR. Capture **`commentId`** from the response for `focusedCommentId`.

4. **Update Bitbucket PR description:** prepend one line (or short block) with the **focused comment URL** (see PR description prepend below) and optionally the TymeX guideline link. Preserve existing bullet list. Use `updatePullRequest` if only the description changes; title should already be `[KEY] …`.

5. **Keep Jira ticket body honest:** After significant delivery steps, merge **`jira_update_issue`** into the ticket **description** (do not rely only on comments):
   - **Delivery / branch / PR:** active PR URL, branch name, and any **superseded** declined PRs (so someone opening the ticket months later does not follow a dead link).
   - Short pointer under **Explicit test result** that the table lives in a comment + PR links to `focusedCommentId`.

6. **If the PR number changes** (new PR after branch rename, declined duplicate): **`jira_edit_comment`** on the test-result comment **and** any older “opened PR” comments so every Bitbucket URL on the ticket matches the **current** open PR. Update the new PR’s description prepend if `focusedCommentId` is unchanged (URL to Jira stays valid).

7. **Tell the user** to verify “Passed” cells and env/device match real runs.

### When replacing a PR (same commits, new branch or new id)

1. Post or keep the Jira test comment; note `commentId` for `?focusedCommentId=`.
2. **`declinePullRequest`** (old) → **`createPullRequest`** (new source branch or fresh PR) — you cannot retarget source branch via API.
3. **`jira_edit_comment`**: set **Pull request:** to the **new** PR HTML URL.
4. **`updatePullRequest`** on the **new** PR: prepend focused Jira test URL (step 4 above).
5. **`jira_update_issue`**: description **Delivery** section lists active vs superseded PRs.

## Jira comment template (Markdown)

Adapt table rows to the feature; keep structure aligned with CB-1256.

```markdown
**PR title:** [<KEY>] <short title from Bitbucket>
**Pull request:** <Bitbucket PR HTML URL>

Test result

- Env: <e.g. Local dev / UAT>
- Device: <e.g. Simulator iPhone 15, iOS latest>

| **Cases** | **Result** |
| --- | --- |
| <Case 1> | Passed |
| <Case 2> | Passed |

#testresult
```

## PR description prepend (Markdown)

```markdown
**Jira test result (explicit test result per [TymeX guideline](https://ambcba.atlassian.net/wiki/spaces/TM/pages/3737190876)):** https://ambcba.atlassian.net/browse/<KEY>?focusedCommentId=<commentId>

```

Then append a blank line and the **existing** PR body unchanged.

## Jira description scope (wiki-style example)

If the ticket uses Jira wiki markup:

```text
h2. Scope

* <bullet tied to PR>
* <bullet>

----

h2. Design

<existing screenshot / Figma link>
```

If the instance expects Markdown for descriptions, mirror the same sections in Markdown.

## Edge cases

- **No tests apply:** Post a single comment: `Test result: None` (per guideline).
- **Cannot access Bitbucket:** Ask for PR title and list of changes; still post Jira comment without PR link or with a pasted title.
- **Read-only MCP:** Output the table and links for the user to paste manually; do not claim tools ran.
- **Stale comments:** Any automated or manual comment that says “Pull request: …/pull-requests/346” must be **edited** after 346 is declined — reviewers often read the first PR comment, not the latest test table.

## Anti-patterns

- Do not mark **Passed** without user confirmation when they have not stated tests were run — either ask or label rows as pending.
- Do not strip existing Jira attachments or design links when updating description; prepend **Scope** or merge carefully.
