---
name: codeql
description: "CodeQL deep security scanning with interprocedural data flow and taint tracking. Use when running CodeQL analysis, building CodeQL databases, creating custom CodeQL queries, or performing thorough security audits that require cross-function data flow analysis."
---

# CodeQL Analysis

Performs deep security scanning using CodeQL's interprocedural data flow and taint tracking analysis.

## Supported Languages

Python, JavaScript/TypeScript, Go, Java/Kotlin, C/C++, C#, Ruby, Swift

## Three Core Workflows

| Workflow | Purpose |
|----------|---------|
| `build-database` | Create CodeQL database via sequential build methods |
| `create-data-extensions` | Generate source/sink models for project-specific APIs |
| `run-analysis` | Execute queries and process results |

## Key Principles

**Database quality matters:** A database that builds is not automatically good. File counts and extractor errors must be verified — a cached build can produce zero useful extraction.

**Custom models are essential:** Even standard frameworks like Django or Spring often have custom wrappers. Skipping the `create-data-extensions` workflow means missing vulnerabilities in project-specific code paths.

**Always use explicit suite references:** Passing pack names directly to `codeql database analyze` applies hidden filters that can produce zero results.

**Zero findings ≠ clean code:** Investigate before reporting — the cause may be poor database quality, missing models, or incorrect query packs.

## Output Structure

All artifacts land in a single `$OUTPUT_DIR` (auto-incremented as `static_analysis_codeql_1`, `_2`, etc.):

```
$OUTPUT_DIR/
├── codeql.db/          # Database
├── extensions/         # Data extension YAMLs
├── raw/results.sarif   # Unfiltered output
└── results/results.sarif  # Final results
```

## Common Commands

```bash
# Build database
codeql database create codeql.db --language=python --source-root=.

# Run analysis with explicit suite
codeql database analyze codeql.db python-security-and-quality.qls \
  --format=sarif-latest --output=results.sarif

# Validate database
codeql database info codeql.db
```

## macOS Apple Silicon Note

Exit code 137 signals an `arm64e`/`arm64` mismatch — not a build failure. Try Homebrew arm64 tools or Rosetta before falling back to `build-mode=none`, which produces severely incomplete analysis.

## Pitfalls

- Never rely on zero findings without first verifying database extraction quality
- Custom framework wrappers require data extensions — check before concluding clean
- Use `--suite-file` or explicit `.qls` references, not just pack names

Source: https://github.com/trailofbits/skills
