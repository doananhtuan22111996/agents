---
name: firebase-apk-scanner
description: "Scans Android APKs for Firebase misconfigurations — open databases, exposed storage, auth weaknesses. Use when analyzing Android APKs for Firebase security issues. Requires explicit authorization. Android only (not iOS/web)."
---

# Firebase APK Security Scanner

Scans Android APKs to detect Firebase security issues.

## Core Purpose

Detects Firebase security issues including:
- Open/unauthenticated databases (Realtime DB + Firestore)
- Exposed storage buckets
- Authentication weaknesses
- Accessible Cloud Functions
- Remote Config exposure

## Workflow Summary

1. **Validate** the target APK/directory exists
2. **Run** the bundled `scanner.sh` which decompiles via apktool and extracts Firebase config
3. **Test** all discovered endpoints (auth, database, storage, functions)
4. **Report** findings by severity

## Severity Levels

| Level | Example Finding |
|-------|----------------|
| CRITICAL | Unauthenticated DB read/write |
| HIGH | Anonymous auth enabled, bucket listing |
| MEDIUM | Email enumeration, exposed functions |
| LOW | Info disclosure without sensitive data |

## Rationalizations to Reject

- **"The database is read-only so it's fine"** → Data exposure remains a critical finding regardless
- **"It's protected by app-level auth"** → Firebase rules must enforce auth independently
- **"Only internal users can access it"** → APK config is readable by anyone who decompiles it

## Scope Limitations

- Android APKs only (not iOS/web)
- Requires **explicit written authorization** to test
- Not for production scanning without written permission

## Prerequisites

- `apktool` installed
- `curl` available
- Written authorization from app owner

## Usage

```bash
# Decompile APK
apktool d target.apk -o decompiled/

# Extract Firebase config
grep -r "google-services" decompiled/
grep -r "firebase" decompiled/ --include="*.xml"
```

Source: https://github.com/trailofbits/skills
