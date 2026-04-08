---
name: ios-auto-verify
description: "iOS: Auto-verify with Maestro — generates Maestro UI test flows from manual test cases, runs them on iOS Simulator with video recording, and uploads recordings to Jira. Use after /ios-verify generates manual test cases. Triggers on: auto verify, Maestro test, UI recording, simulator recording, video proof, auto-verify Jira."
---

# iOS: Auto-Verify with Maestro (simulator recording + Jira upload)

$ARGUMENTS

Generate Maestro UI test flows from manual test cases, run them on the simulator with video recording, and upload recordings to Jira. This is best-effort — it should never block the ticket workflow.

## Prerequisites

- **Maestro CLI** installed (`~/.maestro/bin/maestro`) — if missing: `curl -fsSL "https://get.maestro.mobile.dev" | bash`
- **Main app built & installed** on simulator (from `/ios-verify` Step 3)
- **Simulator booted** with the main app available
- **Manual test cases** generated (from `/ios-verify` Step 6)
- **Deeplink** to the feature — from ticket, repo `CLAUDE.md`, or ask user. If unavailable, skip auto-verify.

## Safety

- Never print or log Jira API tokens, Slack bot tokens, or test account credentials
- Only upload to the specific Jira ticket in the workflow
- Only interact with the local simulator — never push to remote devices
- Always clean up temp files after upload (Step 8) — recordings are 5-15 MB each

## Environment config (single reference)

All environment-specific values in one place. Default to **SIT** unless the ticket specifies otherwise. GTZA/SLZA have no DEV environment.

| Target | Repo | Bundle ID (SIT) | Bundle ID (UAT) | URL Scheme (SIT) | URL Scheme (UAT) | Scheme (SIT) | Passcode |
|--------|------|----------------|-----------------|-------------------|-------------------|-------------|----------|
| GTZA | `tb-gosa-ios-app` | `za.co.gotyme.sit` | `za.co.gotyme.uat` | `gtzasit://` | `gtzauat://` | `MainAppSIT` | `1357` |
| SLZA | `tb-sanlam-ios-app` | `com.tymex.slza.sit` | `com.tymex.slza.uat` | `slzasit://` | `slzauat://` | `MainAppSIT` | `1357` |
| GTPH | `ph-goph-ios-app` | `ph.com.gotyme.sit` | `ph.com.gotyme.uat` | `gotymesit://` | `gotymeuat://` | `MainAppSIT` | `112233` |

**Deeplink format**: `{URL_SCHEME}{feature_path}` (e.g., `gtzasit://transfer`, `gotymesit://accounts/selection`)

The feature path should be documented in the sub-module repo's `CLAUDE.md`.

## Step 1: Detect simulator and app bundle ID

```bash
export PATH="$PATH:$HOME/.maestro/bin"

DEVICE_UDID=$(xcrun simctl list devices booted -j | python3 -c "
import json, sys
data = json.load(sys.stdin)
for runtime, devices in data.get('devices', {}).items():
    for d in devices:
        if d.get('state') == 'Booted':
            print(d['udid']); sys.exit(0)
" | head -1)

echo "Simulator UDID: $DEVICE_UDID"
```

Look up `APP_BUNDLE_ID` from the environment config table above. Verify the app is installed:

```bash
xcrun simctl listapps "$DEVICE_UDID" 2>&1 | grep "$APP_BUNDLE_ID" > /dev/null
if [ $? -ne 0 ]; then
    echo "WARNING: $APP_BUNDLE_ID not installed. Skipping auto-verify."
    exit 0
fi
```

## Step 2: Create temp directory

```bash
MAESTRO_DIR="/tmp/maestro-verify-$(date +%s)"
mkdir -p "$MAESTRO_DIR/flows" "$MAESTRO_DIR/recordings"
```

## Step 3: Login flow (if needed)

The app must be on the landing page before running test cases.

**Scenario A — Already linked**: App shows home screen after launch → skip to Step 4.

**Scenario B — Passcode screen**: Device is linked but locked. Use passcode from env config table.

```yaml
# login.yaml — Do NOT use launchApp (app already launched via simctl, deeplink queued)
appId: {APP_BUNDLE_ID}
---
- extendedWaitUntil:
    visible: "Reset Now"
    timeout: 30000

# Enter passcode (GTZA/SLZA: 1,3,5,7 — GTPH: 1,1,2,2,3,3)
- tapOn: "1"
- tapOn: "3"
- tapOn: "5"
- tapOn: "7"

- waitForAnimationToEnd
- waitForAnimationToEnd

# Dismiss ads/promo popup
- tapOn:
    text: ".*[Cc]lose.*"
    optional: true
- tapOn:
    id: "close"
    optional: true

- waitForAnimationToEnd
```

**Scenario C — Link Device screen**: Full onboarding needed. Requires test account + OTP from Slack.

Test accounts: look up from Confluence based on target/env:
- GTZA/SLZA: [TymeBank Payment Authorization accounts](https://ambcba.atlassian.net/wiki/spaces/CPP/pages/4279665138)
- GTPH: Search Confluence for `SAIYAN - GoPH - {ENV} Test Accounts`

OTP channels: GTPH → `C024GGLMP2B` | GTZA/SLZA → `C080G9YUNGK`

Link-device flow:
1. `launchApp` → tap "Log in" → tap "Let's do it" → enter account ID → enter passcode
2. At OTP screen: take screenshot → read reference number → fetch OTP from Slack via `mcp__slack__slack_get_channel_history` → enter OTP digits
3. Tap "Log in" → dismiss popup → app reaches landing page

The OTP step requires **splitting into separate Maestro flow segments** with Slack MCP calls between them.

## Step 4: Launch app + deeplink (BEFORE login)

The correct order is **launch → deeplink → passcode**. The app queues the deeplink and auto-navigates after login.

Why this specific order: if you open a deeplink while the app is NOT running, iOS shows a "Open in {AppName}?" confirmation dialog that blocks automation. Launching first avoids this. Also, never use Maestro's `openLink` — it goes through Safari and always triggers the dialog.

```bash
xcrun simctl terminate "$DEVICE_UDID" "$APP_BUNDLE_ID" 2>/dev/null; sleep 1
xcrun simctl launch "$DEVICE_UDID" "$APP_BUNDLE_ID"; sleep 2
xcrun simctl openurl "$DEVICE_UDID" "${URL_SCHEME}${FEATURE_PATH}"
```

Then Maestro picks up from the passcode screen (do NOT use `launchApp` in Maestro — app is already running).

## Step 5: Generate Maestro flows from manual test cases

For each test case from `/ios-verify` Step 6, create a Maestro YAML flow.

**Conversion reference:**

| Manual Step | Maestro Command |
|-------------|-----------------|
| Launch app | Handled via `xcrun simctl` (Step 4) — never use `launchApp` |
| Tap "X" | `- tapOn: "X"` or `- tapOn: { id: "accessibility_id" }` |
| Enter text | `- tapOn: "field"` then `- inputText: "value"` |
| Scroll to find "X" | `- scrollUntilVisible: "X"` |
| Verify visible | `- assertVisible: "X"` |
| Verify not visible | `- assertNotVisible: "X"` |
| Wait for loading | `- waitForAnimationToEnd` or `- extendedWaitUntil: { visible: "X", timeout: 10000 }` |
| Swipe | `- swipe: { direction: LEFT }` |
| Go back | `- back` |
| Screenshot | `- takeScreenshot: "name"` |

**Flow template:**

```yaml
# TC-{N}: {Title}
appId: {APP_BUNDLE_ID}
tags: [verify, PNL-{TICKET_ID}]
---
- startRecording: TC-{N}
- waitForAnimationToEnd
# {test steps here}
- takeScreenshot: "TC-{N}-final"
- stopRecording
```

**Guidelines:**
- Prefer text selectors over accessibility IDs (we can't inspect the app)
- Add `waitForAnimationToEnd` after navigation transitions
- Use `scrollUntilVisible` before asserting off-screen elements
- Use `extendedWaitUntil` (10-15s) for network-dependent screens
- Never use `clearState: true` — we need the logged-in state
- One flow per test case, always wrap with `startRecording`/`stopRecording`
- For ambiguous assertions, use `takeScreenshot` instead — screenshot is evidence for manual review
- Before each flow: `xcrun simctl openurl` to deeplink back (reset nav state)

Write flows to `$MAESTRO_DIR/flows/TC-{N}.yaml`.

## Step 6: Run Maestro tests

`startRecording` saves MP4 to the current working directory, so cd into recordings first:

```bash
export PATH="$PATH:$HOME/.maestro/bin"
export MAESTRO_CLI_NO_ANALYTICS=true
export MAESTRO_CLI_ANALYSIS_NOTIFICATION_DISABLED=true

cd "$MAESTRO_DIR/recordings"

for flow in "$MAESTRO_DIR/flows/"*.yaml; do
    tc_name=$(basename "$flow" .yaml)
    echo "=== Running $tc_name ==="

    xcrun simctl terminate "$DEVICE_UDID" "$APP_BUNDLE_ID" 2>/dev/null; sleep 1
    xcrun simctl launch "$DEVICE_UDID" "$APP_BUNDLE_ID"; sleep 2
    xcrun simctl openurl "$DEVICE_UDID" "{DEEPLINK_URL}" 2>/dev/null; sleep 1

    maestro --device "$DEVICE_UDID" test \
        --format junit \
        --output "$MAESTRO_DIR/recordings/${tc_name}-report.xml" \
        "$flow" 2>&1 | tee "$MAESTRO_DIR/recordings/${tc_name}.log"

    echo "Result: $?"
done

cd -
```

**If a flow fails**: check the log, fix the YAML (common: wrong selector → try different text; timeout → increase `extendedWaitUntil`), re-run just that flow. Never block the whole workflow — record partial results.

**Fallback** if Maestro `startRecording` has issues — use `xcrun simctl io` screen recording:

```bash
xcrun simctl io "$DEVICE_UDID" recordVideo --codec h264 "$MAESTRO_DIR/recordings/full-verify.mp4" &
RECORD_PID=$!
# Run all flows, then:
kill -SIGINT $RECORD_PID 2>/dev/null; wait $RECORD_PID 2>/dev/null
```

## Step 7: Upload recordings to Jira

Upload each MP4 as a Jira attachment, then add a comment referencing them.

**7a. Upload attachments** — use the Jira REST API:
- Read credentials from `~/.claude.json` → `mcpServers.jira.env` (ATLASSIAN_JIRA_BASE_URL, EMAIL, API_TOKEN)
- POST each MP4 to `/rest/api/3/issue/{TICKET_KEY}/attachments` with `multipart/form-data`
- Set header `X-Atlassian-Token: no-check` (required for attachment uploads)
- Auth: Basic (email:token base64-encoded)

**7b. Add comment** with test results (ADF format via `/jira` skill approach):

```
## Auto-Verification Results (Maestro)

| Test Case | Result | Recording |
|-----------|--------|-----------|
| TC-1: {title} | PASSED | [TC-1.mp4] |
| TC-2: {title} | FAILED | [TC-2.mp4] |

Environment: {iPhone model} (iOS {version}), {APP_BUNDLE_ID}, Deeplink: {DEEPLINK_URL}
Notes: {failures or manual verification needed}
```

## Step 8: Cleanup

Maestro recordings are 5-15 MB each — cleanup prevents disk bloat from repeated runs.

```bash
rm -rf "$MAESTRO_DIR"
ls -dt ~/.maestro/tests/*/ 2>/dev/null | tail -n +6 | xargs rm -rf 2>/dev/null
```

## Step 9: Log output

```
=== [ios-auto-verify] ===
Status: PASSED | PARTIAL | FAILED | SKIPPED
Simulator: {model} ({UDID})
App: {bundle_id} | Env: SIT | UAT | DEV
Login: passcode | link-device | already-logged-in
Deeplink: {url} | NONE (skipped)
Flows: {generated} generated, {passed} passed, {failed} failed
Recordings: {count} MP4 → {count} uploaded to {TICKET_KEY}
Jira comment: POSTED | FAILED
Cleanup: DONE
```

## When to skip

Skip (not fail) when:
1. **No deeplink** — ticket and repo CLAUDE.md don't specify one. Ask user; if not provided, skip.
2. **Complex preconditions** — feature needs specific account state that can't be set up via Maestro. Take screenshots only.
3. **Link device fails** — OTP not received, Slack unreachable. Note and skip.

Auto-verify never blocks the ticket workflow. If skipped, `/ios-ticket` continues normally.
