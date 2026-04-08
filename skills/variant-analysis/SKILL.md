---
name: variant-analysis
description: "Pattern-based cross-codebase vulnerability search — finds similar instances of a known bug. Use AFTER discovering a vulnerability when you need to hunt for the same root cause elsewhere in the codebase. Not for initial discovery."
---

# Variant Analysis

Systematic approach for finding similar vulnerabilities across codebases after an initial issue is discovered.

## Core Purpose

Use this when you've already found a bug and need to hunt for similar instances — **not** for initial discovery or writing fixes.

## The Five-Step Process

1. **Understand the root cause** — not just symptoms, but *why* it's vulnerable
2. **Create an exact match** — your first pattern should hit only the known instance
3. **Identify abstraction points** — variable names can always flex; literal values only sometimes
4. **Iteratively generalize** — change ONE element at a time, reviewing all new matches each step
5. **Triage results** — classify by confidence, exploitability, and priority

> Stop generalizing when false positives exceed ~50%

## Tool Selection

| Need | Tool |
|------|------|
| Fast surface scan | ripgrep |
| Simple patterns | Semgrep |
| Data flow / taint | Semgrep or CodeQL |
| Cross-function analysis | CodeQL |

## Critical Pitfalls

- **Narrow scope**: Always search the *entire* codebase, not just where the original bug lived
- **Over-specific patterns**: Related constructs (e.g., `isAdmin`, `isVerified`) may harbor the same flaw
- **Single manifestation**: One root cause often appears as null-equality bypasses, inverted conditionals, or doc/code mismatches
- **Missing edge cases**: Test with null values, unauthenticated states, and empty collections

## Variant Report Template

```markdown
## Variant Analysis Report

**Original Finding:** [Location + description]
**Root Cause:** [Why it's vulnerable]

### Confirmed Variants
| # | Location | Confidence | Exploitability |
|---|----------|-----------|----------------|
| 1 | file.go:42 | High | Direct |

### Patterns Used
1. Exact: `[pattern]` — matched [N] locations
2. Generalized: `[pattern]` — matched [N] locations, [X] FP
```

Source: https://github.com/trailofbits/skills
