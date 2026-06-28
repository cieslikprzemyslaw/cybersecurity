# GitHub Actions Hardening

## Purpose

This module documents a practical CI/CD security sprint completed on the AppSec Report Builder project.

The goal was to move from a normal validation workflow to a layered security-aware pipeline.

The final pipeline included:

```text
Validate
Docker Runtime
Dependency Security
CodeQL / SAST
Repository Protection
Secrets and Repository Hygiene
```

## Evidence Summary

The full PR list and final CI status are kept in [04 Evidence](04-evidence.md). The final screenshot is stored at [ci-cd-security-github-actions-success.png](../../../assets/ci-cd-security-github-actions-success.png).

The evidence shows a successful `ci.yml` run after merge, including validation, Docker runtime checks and dependency security checks.

## Files in this module

| File | Purpose |
|---|---|
| [01 Pipeline Mental Model](01-pipeline-mental-model.md) | How to think about CI/CD as part of the attack surface. |
| [02 AppSec Report Builder Lab](02-appsec-report-builder-lab.md) | Practical write-up of the implemented controls. |
| [03 Checklist](03-checklist.md) | Checklist for future CI/CD reviews. |
| [04 Evidence](04-evidence.md) | PR numbers, final status and evidence notes. |

## Main takeaway

A secure CI/CD pipeline is not one tool. It is a chain of small controls that make unsafe changes harder to merge unnoticed.

## References

- OWASP CI/CD Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/CI_CD_Security_Cheat_Sheet.html
- GitHub Actions security hardening: https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions
- GitHub CodeQL documentation: https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning-with-codeql
- GitHub secret scanning: https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning
