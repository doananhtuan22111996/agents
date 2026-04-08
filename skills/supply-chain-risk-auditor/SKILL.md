---
name: supply-chain-risk-auditor
description: "Audits project dependencies for supply chain risk — maintainer health, CVE history, popularity, and threat landscape. Use when told 'audit this project's dependencies', reviewing third-party packages, or assessing open-source supply chain risk."
---

# Supply Chain Risk Auditor

Activates when told **"audit this project's dependencies"** and systematically evaluates supply chain risk.

## Key Risk Criteria

A dependency is marked high-risk for any of these reasons:

| Factor | Concern |
|--------|---------|
| Single/anonymous maintainer | Could be bribed or phished; consider the left-pad incident |
| Unmaintained/archived | Vulnerabilities may go unpatched |
| Low popularity | Fewer users means fewer eyes on the project |
| High-risk features | FFI, deserialization, third-party code execution |
| Past CVEs | Especially high/critical severity relative to project size |
| No security contact | Reporters can't disclose vulnerabilities safely |

## Workflow

1. **Setup** — Creates `.supply-chain-risk-auditor/` workspace; initializes `results.md`
2. **Audit** — For each dependency, evaluates all risk criteria using `gh` CLI for exact data (stars, open issues, etc.)
3. **Post-Audit** — Suggests alternatives, tallies counts per risk factor, writes an executive summary and recommendations

## Important Constraints

- Only reports dependencies with **at least one risk factor** — clean deps are omitted by their absence
- Requires `gh` tool to be installed beforehand
- Does **not** perform active CVE scanning (use `npm audit`, `pip-audit`, etc. separately)
- Stays strictly within the `results-template.md` structure

## Complementary Tools

Run these alongside this skill for full coverage:

```bash
npm audit                  # Node.js known CVEs
pip-audit                  # Python known CVEs
cargo audit                # Rust known CVEs
trivy fs .                 # Multi-language container/filesystem scan
```

Source: https://github.com/trailofbits/skills
