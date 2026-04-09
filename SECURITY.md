# Security Policy

## Supported Versions

Security fixes are applied to the latest version on the `main` branch.

## Reporting a Vulnerability

Please do not open public issues for security vulnerabilities.

Report vulnerabilities by contacting the repository maintainer directly through
GitHub private channels or email, and include:

- A clear description of the issue
- Reproduction steps
- Potential impact
- Any suggested remediation

We will acknowledge receipt promptly, investigate, and coordinate a fix and
disclosure timeline.

## Hardening Notes

This repository follows a security-first automation posture:

- Least-privilege GitHub Actions permissions
- Workflow dependencies pinned to immutable commit SHAs
- Dependency review on pull requests
- Automated dependency updates via Renovate
