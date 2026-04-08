---
name: sharp-edges
description: "Security-focused API and configuration footgun analysis — finds designs where the easy path leads to insecurity. Use when reviewing security-sensitive APIs, library designs, SDK interfaces, or configuration systems for dangerous defaults and footgun patterns."
---

# Sharp Edges Analysis

A security-focused framework for identifying **API and configuration footguns** — designs where the easy path leads to insecurity.

---

## Core Principle

"The pit of success: Secure usage should be the path of least resistance."

If developers must deeply understand cryptography or memorize special rules to avoid vulnerabilities, the design has already failed.

---

## Key Footgun Categories

| Category | Risk Level | Example |
|----------|-----------|---------|
| Algorithm selection | Critical | Accepting `"none"` as a valid JWT algorithm |
| Dangerous defaults | Critical | `timeout=0` silently disabling expiry |
| Primitive vs. semantic APIs | High | Swappable `bytes` params with no type enforcement |
| Configuration cliffs | High | `verify_ssl: fasle` (typo) silently accepted |
| Silent failures | High | Returning `True` when no key is provided |
| Stringly-typed security | Medium | Permissions as comma-separated strings |

---

## Rationalizations to Reject

- **"It's documented"** → Developers miss docs under deadline pressure; make secure choices the *default*
- **"Advanced users need flexibility"** → Most "advanced" usage is copy-paste; provide safe high-level APIs instead
- **"Nobody would actually do that"** → Assume maximum developer confusion

---

## Three Adversary Types to Consider

1. **The Scoundrel** — Can they disable security via config or downgrade algorithms?
2. **The Lazy Developer** — Is the first copy-paste example they find *secure*?
3. **The Confused Developer** — Can they silently swap params or use the wrong key type?

---

## Critical Edge Cases to Probe

For every security-relevant API:
- What does `0`, `""`, `null`, or `-1` actually do?
- Are defaults the *most* secure option, not just a reasonable one?
- Do constructor parameters validate inputs, or merely *default* them?

The distinction matters: a good default still allows dangerous overrides if no validation occurs.

---

## Analysis Checklist

- [ ] Can security be disabled via config flag or env variable?
- [ ] Does the API accept algorithm names as strings (e.g., JWT alg)?
- [ ] Are there silent failure modes that return success when they shouldn't?
- [ ] Can parameters be transposed without a type error?
- [ ] Does typo in a security option silently use a dangerous default?
- [ ] Is the first example in the docs secure?

Source: https://github.com/trailofbits/skills
