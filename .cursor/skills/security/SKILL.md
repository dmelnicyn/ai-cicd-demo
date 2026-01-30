---
name: security
description: Perform static analysis and dependency vulnerability scanning to maintain project security.
---

# Security Skill

The security skill guides security analysis and remediation.

## When to Use

- During development and code review.
- Before merging or releasing.
- Periodically to keep dependencies secure.

## Instructions

- Identify the project’s language and run appropriate vulnerability scanners (e.g., `pip-audit`, `npm audit`, `govulncheck`, `owasp-dependency-check`).
- Analyse dependencies and highlight packages that need updates.
- Run static code analysis tools to detect common security issues (SQL injection, XSS, insecure deserialization).
- Provide a report summarizing findings, severity and remediation steps.
- Update dependencies or code to fix vulnerabilities, verifying that tests still pass.
- Document security considerations and update `SECURITY.md` if necessary.