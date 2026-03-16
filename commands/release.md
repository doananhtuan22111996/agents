# /release — Release Notes
 
Generate release notes for: **$ARGUMENTS**
(e.g., "v1.2.0" or "v1.2.0, closed beta, Task Tracker")
 
## Instructions
 
Look at the git log since the last tag, then generate polished release notes in two formats:
 
```
git log <last-tag>...HEAD --oneline
git tag --sort=-version:refname | head -5
```
 
---
 
## Internal Release Notes (for Notion / personal record)
 
**Title**: [Project] Release — [version]
**Date**: [today]
**Distribution**: Internal / Closed Beta / Public
**Build**: [version code]
**Min SDK**: [from project]
 
### What's New
- Feature/improvement descriptions (user-facing language)
 
### Bug Fixes
- Bug descriptions (what the user experienced, what's fixed)
 
### Internal Changes
- Refactors, dependency updates, infra changes (not shown to users)
 
### Known Issues
- Any known bugs or limitations in this release
 
### Testing Notes
- How was this release tested before shipping?
 
---
 
## Play Store / Beta Release Notes (max 500 chars, user-facing)
 
Write in plain language. No technical jargon. Focus on user value.
 
```
What's new in [version]:
 
• [Feature 1 in one line]
• [Feature 2 in one line]
• [Bug fix in user terms]
```
 
---
 
## Version Bump Checklist
- [ ] `versionCode` incremented in `build.gradle.kts`
- [ ] `versionName` updated (`MAJOR.MINOR.PATCH`)
- [ ] Git tag created: `git tag -a v[version] -m "Release v[version]"`
- [ ] CHANGELOG.md updated (if maintained)
- [ ] Release notes saved to Notion
- [ ] APK / AAB built with release signing config
- [ ] Uploaded to Play Console (Internal Testing / Closed Beta / Production)
