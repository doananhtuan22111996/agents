---
name: semgrep-rule-creator
description: "Creates custom Semgrep rules for detecting security vulnerabilities, bug patterns, and enforcing coding standards. Use when asked to write a new Semgrep rule, detect a specific vulnerability pattern, or create SAST checks. Not for running existing rulesets."
---

# Semgrep Rule Creator

Creates custom Semgrep rules for detecting security vulnerabilities, bug patterns, and enforcing coding standards. **Not** for running existing rulesets or general static analysis.

---

## Core Approach

**Two detection strategies:**
- **Taint mode** (preferred): Tracks data flow from untrusted sources to dangerous sinks — reduces false positives for injection-style vulnerabilities
- **Pattern matching**: For straightforward syntactic patterns with no data flow requirements

Switching between approaches mid-development is acceptable; the goal is a working, precise rule.

---

## Mandatory Workflow (all steps required)

1. Analyze the problem
2. Write tests *first*
3. Analyze AST structure
4. Write the rule
5. Iterate until `semgrep --test` passes 100%
6. Optimize (only after all tests pass)
7. Final run

---

## Key Rules & Constraints

- **One YAML file = one rule** — never bundle multiple rules together
- **No `generic` language** when targeting a specific language
- **Forbidden annotations**: `todoruleid` and `todook` are not permitted in test files
- Tests must cover both vulnerable *and* safe cases

---

## Output Structure

```
<rule-id>/
├── <rule-id>.yaml      # The rule
└── <rule-id>.<ext>     # Test file with annotations
```

---

## Rule Template

```yaml
rules:
  - id: my-rule-id
    message: Description of the vulnerability
    severity: ERROR  # ERROR, WARNING, INFO
    languages: [python]
    metadata:
      cwe: CWE-89
      owasp: A03:2021
    pattern: |
      # pattern or taint config here
```

## Test File Annotations

```python
# ruleid: my-rule-id
vulnerable_code()

# ok: my-rule-id
safe_code()
```

---

## Common Pitfalls

| Anti-Pattern | Problem |
|---|---|
| `pattern: $FUNC(...)` | Matches everything — useless |
| Only testing the vulnerable case | Misses false positives |
| Overly literal patterns | Misses code variations |
| "Pattern looks complete" | Still run `--test` — visual review is insufficient |

## Validate & Test

```bash
semgrep --validate --config rule.yaml
semgrep --test --config rule.yaml test-file
```

Source: https://github.com/trailofbits/skills
