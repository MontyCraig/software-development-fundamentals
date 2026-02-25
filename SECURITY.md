# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in this project, please report it responsibly.

**Do not open a public issue.** Instead, email the maintainer directly at the address
listed in the repository owner's GitHub profile.

### What to include

- A description of the vulnerability
- Steps to reproduce
- The potential impact
- Any suggested remediation (optional)

### Response timeline

- **Acknowledgment**: within 48 hours
- **Initial assessment**: within 7 days
- **Resolution or mitigation**: within 30 days for confirmed issues

### Scope

This repository contains educational reference material (Markdown files with pseudocode
and ASCII diagrams). Security concerns are most likely to involve:

- CI/CD pipeline configuration (GitHub Actions workflow)
- Supply chain risks (pinned action versions, dependency management)
- Accidental inclusion of secrets or credentials

### Supported Versions

| Version | Supported |
|---------|-----------|
| Latest (`main` branch) | Yes |
| Older branches | No |

## Security Practices

This project follows these security practices:

1. **GitHub Actions pinned to SHA** — all workflow actions reference full commit SHAs,
   not mutable tags, to prevent supply chain attacks.
2. **Dependabot enabled** — automated dependency updates for GitHub Actions.
3. **Branch protection** — the `main` branch is protected by repository rulesets
   requiring pull request reviews.
4. **No executable code** — this is a documentation-only repository. No application
   code, no dependencies beyond CI tooling.
