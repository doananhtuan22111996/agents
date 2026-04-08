---
name: sarif-parsing
description: "SARIF (Static Analysis Results Interchange Format) parsing best practices. Use when processing CodeQL or Semgrep output files, aggregating static analysis results, deduplicating findings, or integrating SARIF into CI/CD pipelines."
---

# SARIF Parsing Best Practices

Handles reading, analyzing, and processing SARIF (Static Analysis Results Interchange Format) files from tools like CodeQL and Semgrep.

---

## Core Structure

Every SARIF 2.1.0 file follows this hierarchy:
- **sarifLog** → **runs[]** → **tool** + **results[]** + **artifacts[]**
- Results contain: `ruleId`, `level`, `message.text`, `locations[]`, and `fingerprints{}`

---

## Tool Selection

| Use Case | Tool |
|----------|------|
| Quick CLI queries | `jq` |
| Python (simple) | `pysarif` |
| Python (advanced) | `sarif-tools` |
| Large files (100MB+) | `ijson` |

---

## Key Strategies

### Quick jq Query

```bash
jq '[.runs[].results[]] | length' results.sarif
```

### Python Extraction

Use defensive access since many fields are optional:

```python
loc = result.get("locations", [{}])[0]
phys = loc.get("physicalLocation", {})
```

### Deduplication

Fingerprinting is critical — without stable fingerprints, you can't track findings across runs. When fingerprints are absent, fall back to hashing `ruleId + uri + startLine`.

---

## Critical Pitfalls

1. **Path normalization** — strip `file://`, URL-decode, resolve relative paths early
2. **Fingerprint mismatches** — caused by environment differences or reformatting
3. **Large file performance** — stream with `ijson` instead of loading entirely
4. **Schema validation** — validate before processing to catch malformed files

---

## CI/CD Pattern

```bash
HIGH_COUNT=$(jq '[.runs[].results[] | select(.level == "error")] | length' results.sarif)
# Fail pipeline if HIGH_COUNT > 0
```

---

## Core Principles

- Validate structure first
- Normalize paths immediately
- Combine multiple fingerprint strategies
- Preserve tool metadata when aggregating

Source: https://github.com/trailofbits/skills
