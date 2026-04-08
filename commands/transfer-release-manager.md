---
name: transfer-release-manager
description: Transfer Release Manager
---

# Transfer Release Manager

Monitor mobile release train channels and flag updates relevant to Transfer-related teams.

## Channel Name to ID Mapping

| Channel Name | Channel ID |
|---|---|
| #goza-mobile-release-train | C0800DJUW79 |
| #ph-mobile-release-train | C02BZP0BKBN |
| #slsa-mobile-release-train | C085ZRM3BJ5 |

## Teams to Watch

Flag any messages that mention or relate to these teams/squads:

| Team | Common Keywords |
|---|---|
| Mochi | mochi |
| Saiyan | saiyan |
| Pluto | pluto |
| Pryme | pryme |
| Mobile Transfer | mobile transfer, transfer |

## Execution Flow

When the user invokes `/transfer-release-manager`, follow these steps:

### 1. Fetch Recent Messages and Save to Temp Files

For **each** channel in the mapping table, call `mcp__slack__slack_get_channel_history` with `limit: 30`.

- If a channel returns `not_in_channel` error, report it clearly:
  > "Cannot read **#channel-name** — the Slack bot is not a member. Please invite the bot to the channel and retry."
- Continue fetching the remaining channels even if one fails.

After fetching, **save** each channel's raw JSON result to temp files:
- `/tmp/release-train-goza.json`
- `/tmp/release-train-goph.json`
- `/tmp/release-train-slsa.json`

Use Bash to write each MCP tool result file to the temp path (the persisted output files from MCP can be copied directly).

### 2. Run Parser Script

Run the Node.js parser to filter, format, and generate the report:

```bash
node /Users/vuongtran/Documents/Projects/mobile-helper-scripts/TeamCityScripts/Automation/susanoo/AISkils/transfer-release-manager/index.js \
  /tmp/release-train-goza.json \
  /tmp/release-train-goph.json \
  /tmp/release-train-slsa.json
```

This script:
- Parses Slack JSON and deduplicates thread broadcasts
- Filters messages by team keywords (Mochi, Saiyan, Pluto, Pryme, Mobile Transfer) and release keywords (versions, RC, rollout, blocker, etc.)
- Outputs a **structured markdown report** to stdout (grouped by channel with team mentions and general updates)
- Outputs **action items** (blockers, PROD testing, rollouts, transfer-team items)
- Writes a `brain-data.json` sidecar file with structured data for office brain sync

The config (channels, teams, keywords) lives in `transfer-release-manager/config.js` — edit there to add/remove teams or channels.

### 3. Present the Report

Output the script's stdout directly to the user. The report is already formatted as:
- Per-channel sections with Team Mentions and General Release Updates
- Slack links for each message
- Action Items checklist at the end

### 4. Review and Enhance Action Items

Review the auto-generated action items from the script. Use your judgment to:
- Remove stale items (e.g., old PROD testing that's clearly done)
- Add context from today's date (e.g., "rollout reaches 100% today")
- Highlight Transfer-specific items that need immediate attention
- Add any deadline-sensitive items the script may have missed

### 5. Check Cronjob Expiration

Before syncing to the office brain, check if the recurring cronjob for this skill is about to expire or missing:

1. Call `CronList` to list all scheduled cron jobs.
2. Look for a job with prompt `/transfer-release-manager`.
3. **If no matching cronjob is found**: Alert the user immediately:
   > "⚠️ **Cronjob not found!** The recurring schedule for `/transfer-release-manager` is missing. Run `/loop 9am /transfer-release-manager` to re-create it."
4. **If a matching cronjob exists**: Durable cron jobs auto-expire after **7 days** from creation. Calculate how many days remain based on today's date and the job's creation context:
   - If **≤ 2 days remain** before the 7-day expiry, warn the user:
     > "⏰ **Cronjob expiring soon!** The `/transfer-release-manager` schedule will expire in ~X day(s). Please re-schedule it by running: `/loop 9am /transfer-release-manager`"
   - If **already expired** (no job found after previously being set up), alert as in step 3.
   - Otherwise, note the cronjob is active (no alert needed).

### 6. Sync to Office Brain

After generating the report, sync findings to the virtual office using MCP `office-brain` tools:

1. **Connect to office** (run in background):
   ```
   npx tsx /Users/vuongtran/Documents/Projects/OfficeAgents/src/cli/index.ts join -n "Transfer Release Manager" -r QA -s "Mobile Transfer" -p 3000
   ```

2. **Update activity**:
   ```
   brain_update_activity { agent_name: "Transfer Release Manager", activity: "WORKING", current_task: "Posting release train summary" }
   ```

3. **Post summary** via `brain_post_summary`:
   - `agent_name`: `Transfer Release Manager`
   - `level`: Use appropriate severity:
     - `info` — normal daily update, no blockers
     - `warning` — deadlines today, issues needing attention
     - `error` — critical blockers, regressions, or failed rollouts
   - Include current version + phase for each hub (GOZA, GOPH, SLSA)
   - Include team mentions and action items
   - Keep it concise but actionable

4. **Post leveled updates** via `brain_post_update` for individual items:
   - Use `info` for progress notes (e.g., "GOZA v1.2.0 at 1% rollout")
   - Use `warning` for items needing attention (e.g., "SLSA PROD testing ends today")
   - Use `error` for critical items (e.g., "Crashes after full rollout", "Blocker found")

5. **Set shared context** via `brain_set_context`:
   - Key: `release-train-status`
   - Value: one-line status per hub (e.g., `GOZA v1.2.0: 1% rollout | GOPH v1.69.0: PROD testing`)
   - Scope: `squad`, Owner: `Mobile Transfer`

6. **Create tasks** via `brain_post_task` for each action item:
   - `role_tag`: `QA`
   - `created_by`: `Transfer Release Manager`
   - Include relevant TQTC ticket IDs in the description

8. **Set idle** when done:
   ```
   brain_update_activity { agent_name: "Transfer Release Manager", activity: "IDLE" }
   ```

## Daily Schedule

This skill is designed to run daily at **9:00 AM** (local time) via Claude Code's cron scheduler.

To set up the schedule, run:
```
/loop 9am /transfer-release-manager
```

Or configure manually with `CronCreate`:
- Cron: `57 8 * * 1-5` (weekdays, ~9am)
- Prompt: `/transfer-release-manager`
- Durable: `true` (survives session restarts)

Note: Durable recurring tasks auto-expire after 7 days. Re-schedule as needed.

## Thread Deep-Dive

If the user asks to dig into a specific message or thread, use `mcp__slack__slack_get_thread_replies` to fetch the full thread and summarize the discussion.

## Usage Examples

Quick check across all channels:
```
/transfer-release-manager
```

User follow-up:
```
"Show me more details on the Pluto blocker in #goza-mobile-release-train"
```

## Safety

- **Read-only on Slack**: This skill only reads channel history. It never posts or reacts to Slack messages.
- **Write to office brain only**: Posts summaries, context, and tasks to the virtual office dashboard.
- Do not expose user IDs or private information — summarize content, don't dump raw messages.
- If a message contains sensitive data (tokens, passwords, PII), redact it in the summary.
