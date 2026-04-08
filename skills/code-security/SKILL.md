---
name: code-security
description: "Security guidelines for writing secure code. Use when writing code, reviewing code for vulnerabilities, or asking about secure coding practices like 'check for SQL injection' or 'review security'. IMPORTANT: Always consult this skill when writing or reviewing any code that handles user input, authentication, file operations, database queries, network requests, cryptography, or infrastructure configuration (Terraform, Kubernetes, Docker, GitHub Actions) — even if the user doesn't explicitly mention security. Also use when users ask to 'review my code', 'check this for bugs', or 'is this safe'."
---

# Code Security Guidelines

A comprehensive framework covering secure coding rules across more than 15 languages, addressing OWASP Top 10, infrastructure hardening, and 28 distinct vulnerability categories.

## Usage Modes

**Proactive** — Automatically surface relevant vulnerabilities based on language and code patterns, without waiting for an explicit security request.

**Reactive** — When security is raised directly, locate the matching rule category and consult the corresponding file under `rules/` for detailed examples.

### Workflow
1. Identify the language and the code's purpose (user input? DB queries? file I/O?)
2. Prioritize Critical and High impact rules first
3. Pull the specific rule file for code-level examples
4. Apply secure patterns or flag vulnerable ones during review

---

## Language Priority Matrix

| Language | Top Rules |
|----------|-----------|
| Python | SQL injection, command injection, path traversal, SSRF, insecure crypto |
| JavaScript/TypeScript | XSS, prototype pollution, code injection, CSRF, insecure transport |
| Java | SQL injection, XXE, insecure deserialization, insecure crypto, SSRF |
| Go | SQL injection, command injection, path traversal, insecure transport |
| C/C++ | Memory safety, unsafe functions, command injection, path traversal |
| Ruby | SQL injection, command injection, code injection, insecure deserialization |
| PHP | SQL injection, XSS, command injection, code injection, path traversal |
| HCL/YAML | Terraform (AWS/Azure/GCP), Kubernetes, Docker, GitHub Actions |

---

## Vulnerability Categories

### Critical
- **SQL Injection** — parameterized queries only; never concatenate input
- **Command Injection** — avoid shell invocation with user data; prefer safe APIs
- **XSS** — escape all output; rely on framework protections
- **XXE** — disable external entity resolution in XML parsers
- **Path Traversal** — validate and sanitize every file path
- **Insecure Deserialization** — never deserialize untrusted data
- **Code Injection** — never pass user input to `eval()` or equivalents
- **Hardcoded Secrets** — use environment variables or a dedicated secret manager
- **Memory Safety** — prevent buffer overflows and use-after-free in C/C++

### High
- **Insecure Crypto** — SHA-256+, AES-256; retire MD5, SHA-1, DES
- **Insecure Transport** — enforce HTTPS; verify certificates
- **SSRF** — validate URLs against an allowlist
- **JWT Issues** — always verify signatures
- **CSRF** — require tokens on all state-changing requests
- **Prototype Pollution** — validate object keys in JavaScript

### Infrastructure
- Terraform (AWS / Azure / GCP) — encryption, least privilege, no public exposure
- Kubernetes — no privileged containers; run as non-root
- Docker — non-root user; pin image versions
- GitHub Actions — avoid script injection; pin action versions by digest

### Medium / Low
- Regex DoS, race conditions, logic correctness, and general best practices

---

## Quick Reference

| Vulnerability | Core Defense |
|---------------|--------------|
| SQL Injection | Parameterized queries |
| XSS | Output encoding |
| Command Injection | Avoid shell; use APIs |
| Path Traversal | Validate paths |
| SSRF | URL allowlists |
| Secrets | Environment variables |
| Crypto | SHA-256 / AES-256 |

Full index with CWE and OWASP references: `rules/_sections.md`

Source: https://github.com/semgrep/skills
