---
name: release-management
description: Prepare and execute a release, including versioning, changelog and deployment.
---

# Release Management Skill

The release management skill defines the steps required to cut a release.

## When to Use

- When features are complete and quality gates are met.
- Before deploying to staging or production.

## Instructions

- Ensure all QA checks pass and no critical vulnerabilities remain.
- Update `CHANGELOG.md` with user‑visible changes, referencing issue numbers when possible.
- Bump version numbers in appropriate files (e.g., `pyproject.toml`, `package.json`).
- Commit and tag the release in version control.
- Trigger the CI/CD pipeline to build and deploy artifacts.
- Verify deployment succeeded and run smoke tests.
- Communicate release notes to stakeholders.