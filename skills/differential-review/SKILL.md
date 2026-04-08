---
name: differential-review
description: "Security-focused git diff analysis — reviews PRs and commits for security regressions, logic flaws, and blast radius. Use when reviewing a PR for security issues, analyzing a security-sensitive diff, or performing risk-based code review."
---

# Differential Security Review

A security-focused code review methodology for PRs, commits, and diffs.

## Core Approach

Reviews are **risk-first**, scaling analysis depth based on codebase size:

| Size | Strategy |
|------|----------|
| <20 files | Deep — read all dependencies |
| 20–200 files | Focused — priority files only |
| 200+ files | Surgical — critical paths only |

## Key Principles

- **Never skip git history** — it reveals regressions
- **Classify by risk, not size** — "Heartbleed was 2 lines"
- **Always generate a report file** — verbal explanations lose findings
- Blast radius must be calculated *quantitatively*, not assumed

## Risk Triggers

- **HIGH**: Auth, crypto, value transfer, validation removal
- **MEDIUM**: Business logic, new public APIs
- **LOW**: Comments, UI, logging

## Immediate Escalation Flags

Stop and investigate if you observe:
- Removed code tied to "security", "CVE", or "fix" commits
- Access control modifiers dropped (e.g., `internal → external`)
- External calls added without checks

## Workflow

```
Pre-Analysis → Triage → Code Analysis → Test Coverage → Blast Radius → Deep Context → Adversarial → Report
```

### Step-by-Step

1. **Pre-Analysis** — Fetch git log, identify risk-classified files
2. **Triage** — Score each changed file by risk level
3. **Code Analysis** — Line-by-line review of HIGH/MEDIUM files
4. **Test Coverage** — Are security-relevant changes tested?
5. **Blast Radius** — How many callers/consumers are affected?
6. **Deep Context** — Cross-reference with related files and history
7. **Adversarial** — What would an attacker exploit in this diff?
8. **Report** — Write findings to a file, never just verbally

## Report Template

```markdown
## Security Review: [PR/Commit]

**Risk Level:** HIGH / MEDIUM / LOW
**Blast Radius:** [N files, N callers]

### Findings
| Severity | Location | Issue |
|----------|----------|-------|
| HIGH | auth/login.go:42 | Removed rate limiting |

### Recommendations
...
```

Source: https://github.com/trailofbits/skills
