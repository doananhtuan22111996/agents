---
name: slack-post-pr-review
description: >-
  Posts a Bitbucket PR review request to Slack with Jira + summary parsed from the
  PR title, using <!subteam^S07341T078A> for the mobile-devs group (plain @mobile-devs
  does not notify). Default channel #mobile-pr-review; resolve others via
  slack_search_channels. Use when the user asks to ping Slack for a PR review,
  mobile-devs, or TymeX Bitbucket → Slack review workflow.
---

# Slack PR review

## When to apply

- User shares a **Bitbucket PR URL** (or workspace + repo slug + PR id) and wants a **Slack post** for review.
- User wants **Jira auto-detected** from the PR when possible.

## TymeX constants (do not guess IDs)


| Role                           | Value                    |
| ------------------------------ | ------------------------ |
| Mobile devs user group mention | `<!subteam^S07341T078A>` |

Plain `@mobile-devs` in API-sent messages **does not** notify the group—always use `<!subteam^S07341T078A>`.

## Channels

| Channel               | Channel ID     | Notes |
| --------------------- | -------------- | ----- |
| `#mobile-pr-review`   | `C073SS2M70Q`  | **Default** for PR review posts. Use this `channel_id` in `slack_send_message` when the user does not name another channel. |

- **Default:** `#mobile-pr-review` → **`C073SS2M70Q`**. If `slack_search_channels` returns no match (bot not in channel, API limits), still use **`C073SS2M70Q`** for the default channel unless the user specifies a different channel.
- **User-specified:** e.g. `#card-culi-chem-gio` — search the **channel name** (without `#`); confirm ID from results before `slack_send_message`.

## MCP tools


| Step                          | Server               | Tool                                                           |
| ----------------------------- | -------------------- | -------------------------------------------------------------- |
| PR title, description, branch | `user-bitbucket`     | `getPullRequest` (`workspace`, `repo_slug`, `pull_request_id`) |
| Channel ID                    | `plugin-slack-slack` | `slack_search_channels`                                        |
| Post                          | `plugin-slack-slack` | `slack_send_message` (`channel_id`, `message`)                 |


Parse Bitbucket URLs: `https://bitbucket.org/<workspace>/<repo>/pull-requests/<id>` (and `/pull-request/` variants).

## Jira key extraction (order)

1. **PR title:** First match of `\[[A-Z][A-Z0-9]+-\d+\]` (e.g. `[PNL-32384]` → `PNL-32384`).
2. **Branch:** `source.branch.name` from `getPullRequest`, e.g. `feature/PNL-32384-…`.
3. **Description:** `browse/PNL-12345` or `[PNL-12345](https://…jira…)` in `description` / `summary.raw`.

If none: use title summary only; say Jira was not detected—do not invent a key.

## Short intro line

1. Start from Bitbucket `title`.
2. Strip leading `\[[^\]]+\]` segments until text remains (removes `[PNL-32384]`, `[iOS]`, etc.).
3. Trim; if empty, use first description sentence or `PR <id>`.
4. Format: `*KEY* — <remainder>` with Slack `*bold`*. No key: `*Review requested* — <remainder>`.

## Message template

Slack markdown; newline before URL:

```text
Hi <!subteam^S07341T078A>, please help me review PR *<KEY>* — <remainder from title>. Thanks mn!
<Bitbucket PR HTML URL>
```

Adjust greeting/closing only if the user asks; keep subteam mention and Jira/link behavior unless they specify another group.

## Workflow checklist

1. Parse URL → `getPullRequest`.
2. Extract Jira key; build remainder from `title`.
3. Resolve channel: user-named channel **or** default `#mobile-pr-review` → **`C073SS2M70Q`**.
4. `slack_send_message` with template body.
5. Reply with **KEY** (or note if missing), **final line**, **Slack message permalink**.

## Merge monitoring (optional follow-up)

Use with the **`slack-pr-merge-monitor`** agent (`~/.agent/agents/slack-pr-merge-monitor.md`) when the user wants a **Slack post for review** and then **notify Slack when the PR merges** (or is declined/superseded).

1. Complete the workflow checklist above first.
2. Poll **`getPullRequest`** every 60–120s until `state` is `MERGED`, `DECLINED`, or `SUPERSEDED`.
3. Post a short outcome message to the **same channel** (use **`thread_ts`** from the first `slack_send_message` response when available so the merge note stays in-thread).
4. Do not call **`mergePullRequest`** unless the user explicitly requests a merge.

## Notes

- No secrets in the message.
- Slack Connect / external shared channels may block posting—report error and suggest an internal channel or DM.
- `thread_ts` only if the user supplies it (thread reply), or when continuing a thread after the first bot message returns a `ts` for merge notifications.

