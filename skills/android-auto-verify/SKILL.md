---
name: android-auto-verify
description: "Android: Auto-verify with Maestro — generates Maestro UI test flows from manual test cases, runs them on Android Emulator with video recording, and uploads recordings to Jira. Use after /android-verify generates manual test cases. Triggers on: auto verify, Maestro test, UI recording, emulator recording, video proof, auto-verify Jira."
---

# Android: Auto-Verify with Maestro (emulator recording + Jira upload)

$ARGUMENTS

Generate Maestro UI test flows from manual test cases, run them on the emulator with video recording, and upload recordings to Jira. This is best-effort — it should never block the ticket workflow.

## Prerequisites

- **Maestro CLI** installed (`~/.maestro/bin/maestro`) — if missing: `curl -fsSL "https://get.maestro.mobile.dev" | bash`
- **Main app built & installed** on emulator (from `/android-verify` Step 3)
- **Emulator booted** with the main app available
- **Manual test cases** generated (from `/android-verify` Step 6)
- **Deeplink** to the feature — from ticket, repo `CLAUDE.md`, or ask user. If unavailable, skip auto-verify.

## Safety

- Never print or log Jira API tokens, Slack bot tokens, or test account credentials
- Only upload to the specific Jira ticket in the workflow
- Only interact with the local emulator — never push to remote devices
- Always clean up temp files after upload (Step 8) — recordings are 5-15 MB each

## Environment config (single reference)

All environment-specific values in one place. Default to **SIT** unless the ticket specifies otherwise. GTZA/SLZA have no DEV environment.

| Target | Repo | App ID (SIT) | App ID (UAT) | Deeplink Scheme (SIT) | Deeplink Scheme (UAT) | Passcode |
|--------|------|-------------|-------------|----------------------|----------------------|----------|
| GTZA | `tb-gosa-android-app` | `za.co.gotyme.sit` | `za.co.gotyme.uat` | `gtzasit://` | `gtzauat://` | `1357` |
| SLZA | `tb-sanlam-android-app` | `com.tymex.slza.sit` | `com.tymex.slza.uat` | `slzasit://` | `slzauat://` | `1357` |
| GTPH | `ph-goph-android-app` | `ph.com.gotyme.sit` | `ph.com.gotyme.uat` | `gotymesit://` | `gotymeuat://` | `112233` |

**Deeplink format**: `{DEEPLINK_SCHEME}{feature_path}` (e.g., `gtzasit://transfer`, `gotymesit://accounts/selection`)

The feature path should be documented in the sub-module repo's `CLAUDE.md`.

## Step 1: Detect emulator and app package

```bash
export PATH="$PATH:$HOME/.maestro/bin"

DEVICE_SERIAL=$(adb devices | grep -E "emulator-[0-9]+" | head -1 | awk '{print $1}')

echo "Emulator: $DEVICE_SERIAL"
```

Look up `APP_PACKAGE` from the environment config table above. Verify the app is installed:

```bash
adb -s "$DEVICE_SERIAL" shell pm list packages | grep "$APP_PACKAGE" > /dev/null
if [ $? -ne 0 ]; then
    echo "WARNING: $APP_PACKAGE not installed. Skipping auto-verify."
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
# login.yaml
appId: {APP_PACKAGE}
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

## Step 4: Launch app + deeplink

```bash
adb -s "$DEVICE_SERIAL" shell am force-stop "$APP_PACKAGE"; sleep 1
adb -s "$DEVICE_SERIAL" shell monkey -p "$APP_PACKAGE" -c android.intent.category.LAUNCHER 1; sleep 2
adb -s "$DEVICE_SERIAL" shell am start -a android.intent.action.VIEW -d "${DEEPLINK_SCHEME}${FEATURE_PATH}" "$APP_PACKAGE"
```

Then Maestro picks up from the passcode screen (do NOT use `launchApp` in Maestro — app is already running).

## Step 5: Generate Maestro flows from manual test cases

For each test case from `/android-verify` Step 6, create a Maestro YAML flow.

**Conversion reference:**

| Manual Step | Maestro Command |
|-------------|-----------------|
| Launch app | Handled via `adb` (Step 4) — never use `launchApp` |
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
appId: {APP_PACKAGE}
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
- Before each flow: `adb shell am start -a android.intent.action.VIEW -d` to deeplink back (reset nav state)

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

    adb -s "$DEVICE_SERIAL" shell am force-stop "$APP_PACKAGE"; sleep 1
    adb -s "$DEVICE_SERIAL" shell monkey -p "$APP_PACKAGE" -c android.intent.category.LAUNCHER 1; sleep 2
    adb -s "$DEVICE_SERIAL" shell am start -a android.intent.action.VIEW -d "{DEEPLINK_URL}" "$APP_PACKAGE" 2>/dev/null; sleep 1

    maestro --device "$DEVICE_SERIAL" test \
        --format junit \
        --output "$MAESTRO_DIR/recordings/${tc_name}-report.xml" \
        "$flow" 2>&1 | tee "$MAESTRO_DIR/recordings/${tc_name}.log"

    echo "Result: $?"
done

cd -
```

**If a flow fails**: check the log, fix the YAML (common: wrong selector → try different text; timeout → increase `extendedWaitUntil`), re-run just that flow. Never block the whole workflow — record partial results.

**Fallback** if Maestro `startRecording` has issues — use `adb screenrecord`:

```bash
adb -s "$DEVICE_SERIAL" shell screenrecord /sdcard/full-verify.mp4 &
RECORD_PID=$!
# Run all flows, then:
kill $RECORD_PID 2>/dev/null; sleep 1
adb -s "$DEVICE_SERIAL" pull /sdcard/full-verify.mp4 "$MAESTRO_DIR/recordings/"
adb -s "$DEVICE_SERIAL" shell rm /sdcard/full-verify.mp4
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

Environment: {Emulator name} (Android {version}), {APP_PACKAGE}, Deeplink: {DEEPLINK_URL}
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
=== [android-auto-verify] ===
Status: PASSED | PARTIAL | FAILED | SKIPPED
Emulator: {name} ({DEVICE_SERIAL})
App: {app_package} | Env: SIT | UAT | DEV
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

Auto-verify never blocks the ticket workflow. If skipped, `/mobile-ticket` continues normally.
