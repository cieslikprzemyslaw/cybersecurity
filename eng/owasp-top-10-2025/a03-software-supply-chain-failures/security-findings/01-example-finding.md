# Example Security Finding: Non-Reproducible npm Installs Increase Supply Chain Risk

## Summary

A supply chain review identified that the project appears to rely on dynamic npm dependency installation in CI/CD without an active committed lockfile strategy.

Several CI jobs use `npm install`, and the review did not identify a normal committed root or frontend lockfile in active use. This can make dependency resolution less deterministic and may allow transitive dependency versions to change between builds without any source code change.

This finding is based on a public open-source practice review and is written as a learning example, not as a vulnerability disclosure against the reviewed project.

## Affected Area

Affected areas include the dependency installation and CI/CD build process.

Examples of relevant areas reviewed:

```text
package.json
frontend/package.json
GitHub Actions workflows
CI dependency installation steps
build/test/package jobs
```

## OWASP Mapping

```text
OWASP Top 10 2025: A03 - Software Supply Chain Failures
```

Related weakness types:

```text
Non-reproducible dependency installation
Uncontrolled dependency resolution
CI/CD supply chain risk
Dependency management weakness
```

## Risk / Impact

Without a committed lockfile and reproducible CI installation strategy, the dependency tree can change over time even when application source code does not change.

Possible impact:

- future builds may install different transitive dependency versions,
- a newly compromised or vulnerable package version may enter the build unexpectedly,
- debugging and incident response become harder,
- tested artifacts may not be easily reproducible,
- dependency changes may be less visible in pull requests,
- build integrity and auditability are weaker.

This does not automatically mean the application is currently compromised. The issue is that the process has weaker controls against unexpected dependency changes.

## Root Cause

The root cause is the lack of a clearly enforced reproducible dependency installation strategy.

Observed contributing factors during the practice review included:

- no active committed root lockfile observed,
- no active committed frontend lockfile observed,
- repeated use of `npm install` in CI,
- dependency version ranges that may resolve differently over time,
- lifecycle scripts that may execute in jobs that use normal `npm install`.

## Evidence

The CI workflow reviewed included dependency installation steps using `npm install` in multiple jobs.

Example pattern:

```yaml
- name: "Install application"
  run: npm install
```

The workflow also included a safer pattern in the lint job:

```bash
npm install --ignore-scripts
cd frontend
npm install --ignore-scripts
```

This shows that some install-time script execution risk is reduced in specific jobs, but normal `npm install` still appears in other CI jobs.

## Positive Controls Observed

The review also found positive supply chain controls:

- some jobs use `npm install --ignore-scripts`,
- many GitHub Actions are pinned to commit SHA,
- SBOM generation is present through CycloneDX,
- Docker publishing is gated to trusted branches and repository context,
- test/build/package workflows are structured.

These controls reduce some supply chain risk, but they do not fully replace lockfile-based reproducibility.

## Expected Secure Behaviour

A stronger supply chain setup should make dependency installation predictable and auditable.

Expected behaviour:

- lockfiles are committed,
- CI uses `npm ci` where possible,
- dependency tree changes are reviewed through lockfile diffs,
- lifecycle scripts are disabled where not required,
- dependency scanning is enabled,
- SBOM generation is part of the release process,
- build artifacts can be traced back to tested source and dependency versions.

## Remediation

Recommended remediation:

1. Commit the appropriate lockfiles, such as `package-lock.json` and `frontend/package-lock.json`, if npm is the selected package manager.
2. Use `npm ci` in CI/CD jobs where lockfiles are available.
3. Review and document any job that must use `npm install` instead of `npm ci`.
4. Continue using `--ignore-scripts` in jobs where lifecycle scripts are not required.
5. Review jobs that run normal `npm install` while secrets are available.
6. Add dependency scanning or ensure repository-level dependency alerts are enabled.
7. Keep SBOM generation in the build/release process.
8. Review lockfile changes during pull requests.
9. Define ownership for dependency update review.
10. Consider artifact provenance/attestation for releases where appropriate.

## Regression Test Ideas

Add CI/CD checks to verify:

```text
Lockfiles are present for npm projects.
CI uses npm ci where lockfiles exist.
CI fails if package.json and package-lock.json are out of sync.
Dependency install jobs do not unexpectedly run lifecycle scripts where not needed.
SBOM generation still runs during package/release.
Dependency scanning runs before release.
Deployment jobs do not install unreviewed dependency versions dynamically.
```

## Developer Takeaway

This review taught me that supply chain security is not only about checking whether a dependency has a known CVE.

For Node.js and frontend projects, AppSec review should also ask:

- Are dependency versions reproducible?
- Can install scripts execute automatically?
- Does CI use lockfiles?
- Are build and release artifacts traceable?
- Are dependency changes visible during review?

A lockfile does not make a project secure by itself, but it is an important control for reproducibility and dependency visibility.

## Learning Reflection

During this review, I also learned about controls that I do not normally see in simple everyday frontend projects.

For example, I had to check what `npm install --ignore-scripts` does. I learned that it can be a useful control because it prevents lifecycle scripts such as `postinstall` from running during installation.

I also saw SBOM generation in practice. I learned that SBOM is a component inventory. It does not fix vulnerabilities by itself, but it helps with vulnerability management and incident response.
