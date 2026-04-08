# iOS: Pre-PR checks (mirrors CI pipeline commitCheck.sh)

Run all iOS-specific checks that the CI pipeline runs on commit. For Sonar scan, this skill delegates to `/mobile-precheck`.

## CI Pipeline Reference

The CI pipeline script is `sonarCloudRunner.sh` (located at `~/Developer/iOS/mobile-helper-scripts/TeamCityScripts/`). Key behaviors:

- **Test scheme**: Derived from xcworkspace name: `basename "$xcworkspace" .xcworkspace` + `Tests` suffix
- **Sonar project key**: `com.tymex:$REPO_NAME` where REPO_NAME is the Bitbucket repo name (e.g., `com.tymex:tc-mx-ios-payment`)
- **Sonar token**: Retrieved from AWS SSM in CI; locally use `$SONAR_TOKEN` from `~/.claude.json` env

## Safety

- **NEVER** print, echo, or log `$SONAR_TOKEN` or any token/key value
- Always pass tokens via env var reference (`$SONAR_TOKEN`), never hardcode

## 1. SwiftLint (strict mode)

```bash
./Pods/SwiftLint/swiftlint lint --strict 2>&1 | grep -v ".claude/"
```

- Filter out `.claude/` violations (not present in CI clone).
- If no violations remain, SwiftLint PASSED.

## 2. Generate SwiftGen and Cuckoo Mocks

```bash
./Pods/SwiftGen/bin/swiftgen
```

If `./script/iosGenerateSwiftGen.sh` exists:
```bash
chmod +x ./script/iosGenerateSwiftGen.sh && ./script/iosGenerateSwiftGen.sh
```

Then:
```bash
bash ./script/iosCuckooGenerateMocks.sh
```

## 3. Detect workspace, test scheme, and simulator

```bash
xcworkspace=$(find . -maxdepth 1 -type d -name "*.xcworkspace" | head -1 | sed 's|^\./||')
moduleName=$(basename "$xcworkspace" .xcworkspace)
TEST_SCHEME="${moduleName}Tests"

xcode_version=$(xcodebuild -version | grep "Xcode" | cut -d' ' -f2)
if [[ "$xcode_version" == "16.2" ]]; then
    iphone_name="iPhone 16"; ios_version="18.3.1"
else
    iphone_name="iPhone 17"; ios_version="26.2"
fi

if [[ -f "tymepipeline.json" ]]; then
    testDest=$(jq -r '.testDestination // empty' tymepipeline.json 2>/dev/null)
    if [[ -n "$testDest" ]]; then
        iphone_name=$(echo "$testDest" | jq -r '.name')
        ios_version=$(echo "$testDest" | jq -r '.OS')
    fi
fi
```

Print detected values for user to verify.

## 4. Build and Run Unit Tests (with code coverage)

```bash
xcodebuild \
  -workspace "$xcworkspace" \
  -scheme "$TEST_SCHEME" \
  -derivedDataPath "build/" \
  -sdk iphonesimulator \
  -destination "platform=iOS Simulator,name=${iphone_name},OS=${ios_version}" \
  -enableCodeCoverage YES \
  test 2>&1 | tee xcodebuild.log
```

Check for `** TEST SUCCEEDED **` or `** TEST FAILED **`. If failed, stop here.

## 5. Coverage extraction

**5a. Find xcresult and check local coverage:**

```bash
xcresult=$(find ./build/Logs/Test -type d -name "*.xcresult" -exec stat -f "%m %N" {} \; | sort -n | tail -n 1 | cut -d ' ' -f 2-)

# Overall coverage
xcrun xccov view --report "$xcresult" --json | jq '(.lineCoverage * 100 * 10 | floor / 10 | tostring) + "%"'

# Per-target
xcrun xccov view --report "$xcresult" --json | jq -r '.targets[] | select(.lineCoverage > 0) | "\(.name): \(.lineCoverage * 100 * 10 | floor / 10)%"'

# Changed files coverage (new code)
changedFiles=$(git diff master --name-only --diff-filter=ACMR | grep '\.swift$' | grep -v 'Tests/' | grep -v 'SwiftGen/')
for file in $changedFiles; do
    filename=$(basename "$file")
    xcrun xccov view --report "$xcresult" --json | jq -r --arg name "$filename" '.targets[].files[] | select(.name == $name) | "\(.name): \(.lineCoverage * 100 * 10 | floor / 10)%"'
done
```

If any changed source file has < 80% coverage, warn that Sonar quality gate may fail.

**5b. Convert xcresult to Sonar coverage XML:**

```bash
echo '<coverage version="1">' > report.xml
xcrun xccov view --archive "$xcresult" | sed -n \
  -e '/:$/s/&/\&amp;/g;s/^\(.*\):$/  <file path="\1">/p' \
  -e 's/^ *\([0-9][0-9]*\): 0.*$/    <lineToCover lineNumber="\1" covered="false"\/>/p' \
  -e 's/^ *\([0-9][0-9]*\): [1-9].*$/    <lineToCover lineNumber="\1" covered="true"\/>/p' \
  -e 's/^$/  <\/file>/p' >> report.xml
echo '</coverage>' >> report.xml
xmlstarlet ed -d '//file[not(contains(@path, ".swift") and substring(@path, string-length(@path)-string-length(".swift")+1)=".swift")]' report.xml > sonarqube-generic-coverage.xml
rm report.xml
```

If `xmlstarlet` is not installed, run `brew install xmlstarlet` first.

## 6. Sonar scan → `/mobile-precheck`

Delegate to the shared Sonar skill with iOS-specific parameters:
- `COVERAGE_REPORT_PATH=sonarqube-generic-coverage.xml`
- `SONAR_EXCLUSIONS=Pods/**,PodLocals/**,build/**,.claude/**,**/GeneratedMocks.swift,**/SwiftGen/**`

## 7. Cleanup

```bash
rm -f xcodebuild.log sonarqube-generic-coverage.xml
```

## 8. Report Results

| Check | Result |
|-------|--------|
| SwiftLint (strict) | PASS/FAIL |
| SwiftGen | PASS/FAIL |
| Cuckoo Mocks | PASS/FAIL |
| Build + Tests | X passed, Y failures / FAIL |
| Coverage (overall) | X% |
| Coverage (new code) | per-file breakdown |
| Sonar Quality Gate | PASS/FAIL (link) |

## 9. Analyze Failures

- If any check fails, analyze the error output and suggest a fix.
- If coverage on new code is < 80%, warn and suggest which files need more tests.
- If Sonar fails, show the dashboard link for details.
- If all checks pass, confirm the branch is ready for PR.

## 10. Log output

```
=== [ios-precheck] ===
Status: PASSED | FAILED
SwiftLint:      PASSED | FAILED (N violations)
SwiftGen:       PASSED | FAILED
Cuckoo Mocks:   PASSED | FAILED
Build + Tests:  PASSED (N passed) | FAILED (N passed, M failed)
Coverage:       overall X% | new code per-file breakdown
Sonar:          PASSED | FAILED (link) | SKIPPED
```
