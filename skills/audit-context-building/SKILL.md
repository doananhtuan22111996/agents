---
name: audit-context-building
description: "Ultra-granular architectural code analysis for pre-vulnerability investigation. Use at the start of a security audit to build deep contextual understanding before any bug-hunting begins. Enforces thorough analysis over surface-level comprehension."
---

# Deep Context Builder

Governs **pre-vulnerability analysis** — building thorough architectural understanding before any bug-hunting begins.

## Core Philosophy

"Slow is fast" — rushed context leads to hallucinated vulnerabilities later. The skill enforces line-by-line analysis over surface-level comprehension.

## Three Phases

### Phase 1: Initial Orientation
- Map modules, entrypoints, actors, and storage
- Do not assume behavior — observe it
- Document trust boundaries and data flows

### Phase 2: Ultra-Granular Function Analysis
Every non-trivial function gets full micro-analysis using:
- **First Principles**: Why does this function exist?
- **5 Whys**: Trace each operation to its root reason
- **5 Hows**: How could this fail or be exploited?

### Phase 3: Global System Understanding
- Reconstruct state invariants
- Map complete workflows end-to-end
- Identify all trust boundaries

## Key Behavioral Rules

- External calls are treated as "adversarial until proven otherwise"
- Call chains are analyzed as **one continuous execution flow** — context never resets
- Contradictions trigger explicit model updates: *"Earlier I thought X; now Y"*
- Vague statements replaced with: *"Unclear; need to inspect X"*

## Quality Thresholds Per Function

- Minimum **3 invariants** documented
- Minimum **5 assumptions** recorded
- Minimum **3 risk considerations** for external interactions

## Hard Boundaries

While active, this skill explicitly prohibits:
- Vulnerability identification
- Fix proposals
- Exploit modeling
- Severity ratings

It exists solely for **deep contextual understanding** — vulnerability analysis comes after this phase completes.

## Context Document Template

```markdown
## Module: [name]

### Purpose
[What this module does and why]

### Actors & Trust Levels
| Actor | Trust | Entry Points |
|-------|-------|-------------|
| User | Untrusted | /api/login |

### State Invariants
1. [invariant]

### External Dependencies
| Dep | Purpose | Trust | Risk |
|-----|---------|-------|------|

### Assumptions
1. [assumption]
```

Source: https://github.com/trailofbits/skills
