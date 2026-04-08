# Mobile: Shared pre-PR checks (Sonar)

This skill contains the **platform-agnostic** pre-PR check steps. It is called by platform-specific skills (`/ios-precheck`, `/android-precheck`) after platform-specific checks (lint, build, tests) have passed.

## Safety

- **NEVER** print, echo, or log `$SONAR_TOKEN` or any token/key value
- Always pass tokens via env var reference (`$SONAR_TOKEN`), never hardcode
- Sonar scanner output goes to `sonar.log` file — do NOT cat the full file (may contain token in URLs). Only grep for PASS/FAIL status
- Only upload coverage data to SonarCloud — no other external service

## 1. Download sonar-scanner (if needed)

```bash
sonarName='sonar-scanner-cli-5.0.1.3006-macosx'
unzipFolder='sonar-scanner-5.0.1.3006-macosx'
if [ ! -f "/tmp/$sonarName.zip" ]; then
    curl -k -L --silent --retry 3 --fail "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/$sonarName.zip" --output "/tmp/$sonarName.zip"
fi
if [ ! -d "/tmp/$unzipFolder" ]; then
    unzip -o -qq "/tmp/$sonarName.zip" -d /tmp/
fi
```

## 2. Detect project key and branch

```bash
REPO_NAME=$(basename -s .git $(git remote get-url origin))
BRANCH_NAME=$(git rev-parse --abbrev-ref HEAD)
SONAR_PROJECT_KEY="com.tymex:$REPO_NAME"
```

## 3. Run sonar-scanner

The caller must provide:
- `COVERAGE_REPORT_PATH` — path to the coverage XML file (format: sonarqube-generic-coverage.xml)
- `SONAR_EXCLUSIONS` — platform-specific exclusions (e.g., `Pods/**,PodLocals/**` for iOS)

```bash
/tmp/sonar-scanner-5.0.1.3006-macosx/bin/sonar-scanner \
  -Dsonar.host.url=https://sonarcloud.io \
  -Dsonar.login="$SONAR_TOKEN" \
  -Dsonar.organization=tymerepos \
  -Dsonar.projectKey="$SONAR_PROJECT_KEY" \
  -Dsonar.c.file.suffixes=- \
  -Dsonar.cpp.file.suffixes=- \
  -Dsonar.objc.file.suffixes=- \
  -Dsonar.sourceEncoding=UTF-8 \
  -Dsonar.qualitygate.wait=true \
  -Dsonar.coverageReportPaths="$COVERAGE_REPORT_PATH" \
  -Dsonar.exclusions="$SONAR_EXCLUSIONS" \
  -Dsonar.branch.name="$BRANCH_NAME" \
  -Dsonar.branch.target=master > sonar.log 2>&1
```

**IMPORTANT:** Use `sonar.exclusions` to skip third-party code — do NOT delete those directories.

## 4. Check result

- If `sonar.log` contains `QUALITY GATE STATUS: PASSED`: Sonar PASSED
- If `sonar.log` contains `FAILURE` or `QUALITY GATE STATUS: FAILED`: Sonar FAILED → show link: `https://sonarcloud.io/dashboard?id=$SONAR_PROJECT_KEY&branch=$BRANCH_NAME`
- Quality gate threshold: **80% coverage on new code**

## 5. Cleanup

```bash
rm -f sonar.log
```

## 6. Results template

```
=== [mobile-precheck: Sonar] ===
Status: PASSED | FAILED
Project key: <key>
Branch: <branch>
Dashboard: <link>
```
