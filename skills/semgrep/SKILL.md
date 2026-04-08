---
name: semgrep
description: "Semgrep static analysis reference — use when running Semgrep scans, writing custom Semgrep rules, interpreting Semgrep findings, or when the user mentions semgrep, SAST, static analysis scanning, or security rule creation."
---

# Semgrep Static Analysis Reference

## Overview
Semgrep is a fast, pattern-based static analysis tool supporting 30+ languages for security scanning and custom rule creation.

---

## MCP Tools (when available)

| Tool | Purpose |
|------|---------|
| `semgrep_scan` | Scan files using built-in rulesets |
| `semgrep_scan_with_custom_rule` | Scan with inline custom YAML rules |
| `semgrep_findings` | Fetch findings from AppSec Platform |
| `semgrep_rule_schema` | Retrieve full rule schema |
| `get_supported_languages` | List supported languages |

---

## Installation

```bash
python3 -m pip install semgrep   # pip
brew install semgrep              # Homebrew
```

---

## Common Rulesets

| Ruleset | Focus |
|---------|-------|
| `p/security-audit` | Comprehensive security |
| `p/owasp-top-ten` | OWASP Top 10 |
| `p/cwe-top-25` | CWE Top 25 |
| `p/trailofbits` | Trail of Bits rules |

---

## Rule Types

**Pattern Matching** — syntactic detection (deprecated APIs, hardcoded values)
**Taint Mode** — data flow from untrusted source → dangerous sink

Prioritize taint mode for injection vulnerabilities. Pattern matching alone can't distinguish between `eval(user_input)` (vulnerable) and a safe literal eval.

### Taint Example

```yaml
mode: taint
pattern-sources:
  - pattern: request.args.get(...)
pattern-sinks:
  - pattern: os.system(...)
```

---

## Key Pattern Syntax

| Syntax | Meaning |
|--------|---------|
| `...` | Match anything |
| `$VAR` | Capture metavariable |
| `<... ...>` | Deep expression match |

---

## Rule Creation Workflow

1. Analyze the bug pattern
2. **Write test cases first** (with `ruleid:` and `ok:` annotations)
3. Dump AST to understand structure
4. Write and iterate on the rule
5. Achieve 100% test pass rate
6. Optimize patterns afterward

---

## Testing

```bash
semgrep --test --config rule.yaml test-file
semgrep --validate --config rule.yaml
```

---

## Critical Reminders

- "Semgrep found nothing" ≠ clean code — it only catches known patterns
- Always include **safe cases** in tests, not just vulnerable ones
- Overly broad patterns (e.g., matching any `$FUNC(...)`) create noise

Source: https://github.com/semgrep/skills
