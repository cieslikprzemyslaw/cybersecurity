# A03:2025 - Software Supply Chain Failures Regression Tests

## Purpose

Regression tests and verification checks for Software Supply Chain Failures should ensure that dependency, build, CI/CD, and release controls do not silently weaken over time.

For A03, “regression tests” are not always traditional unit tests. They can also be CI/CD checks, policy checks, repository rules, dependency scanning, and release verification.

## Test Area 1: Lockfile Presence

### Positive Check

For each npm project in the repository:

```text
package.json exists
package-lock.json exists
```

or the equivalent for the selected package manager:

```text
yarn.lock
pnpm-lock.yaml
```

### Negative Check

The build should fail if:

```text
package.json exists but no approved lockfile exists.
```

### Why This Matters

A lockfile helps ensure the same dependency versions are installed across CI runs.

## Test Area 2: npm ci in CI/CD

### Positive Check

Where a lockfile exists, CI should prefer:

```bash
npm ci
```

instead of:

```bash
npm install
```

### Negative Check

CI should fail or warn when production/test/package jobs use `npm install` without a documented reason.

### Expected Behaviour

```text
npm ci succeeds when package.json and package-lock.json are in sync.
npm ci fails when they are out of sync.
```

## Test Area 3: Lifecycle Script Control

### Positive Check

Jobs that do not require lifecycle scripts should use:

```bash
npm install --ignore-scripts
```

or equivalent package-manager controls.

### Negative Check

CI should flag unexpected use of lifecycle scripts in jobs where they are not needed.

Review these lifecycle scripts:

```text
preinstall
install
postinstall
prepare
```

### Expected Behaviour

```text
Lint/static analysis jobs can install dependencies without running lifecycle scripts.
Jobs that need lifecycle scripts document why they are required.
```

## Test Area 4: Dependency Script Review

### Positive Check

A pull request that adds or changes any of the following should trigger review:

```json
"preinstall": "..."
"install": "..."
"postinstall": "..."
"prepare": "..."
```

### Manual Verification

Review whether the script:

- downloads remote files,
- executes shell commands,
- accesses secrets,
- ignores errors,
- modifies build artifacts,
- runs in CI jobs with sensitive credentials.

## Test Area 5: Dependency Scanning

### Positive Check

A dependency scanning tool should run regularly or on pull requests.

Examples:

```text
Dependabot alerts
npm audit
OSV Scanner
Snyk
Trivy
Grype
GitHub dependency review
```

### Expected Behaviour

High and critical dependency findings should be triaged with an owner and documented decision.

## Test Area 6: SBOM Generation

### Positive Check

The release or package process should generate an SBOM.

Example:

```bash
cyclonedx-npm --omit=dev --output-format=JSON --output-file=bom.json
```

### Negative Check

The release should fail or warn if SBOM generation fails unexpectedly.

### Expected Behaviour

The SBOM should be attached to or included with the release artifact where appropriate.

## Test Area 7: GitHub Actions Pinning

### Positive Check

Third-party GitHub Actions should be pinned to commit SHA where possible.

Example:

```yaml
uses: actions/checkout@<commit-sha>
```

### Negative Check

Flag actions pinned only to mutable tags:

```yaml
uses: some/action@v1
```

### Expected Behaviour

Tag-based actions are reviewed and either accepted with justification or pinned to SHA.

## Test Area 8: Workflow Permissions

### Positive Check

Workflows should use minimal permissions.

Example:

```yaml
permissions:
  contents: read
```

or more specific permissions only where required.

### Negative Check

Flag workflows with broad permissions when not needed.

### Expected Behaviour

Jobs that publish artifacts, push Docker images, deploy to hosting providers, or write security events should have only the permissions they need.

## Test Area 9: Secrets Exposure in CI/CD

### Positive Check

Verify that secrets are available only to jobs that need them.

### Negative Check

Flag jobs where secrets are available while untrusted code or unreviewed dependency scripts can execute.

Examples of risky combinations:

```text
npm install runs lifecycle scripts
+
job has deployment token
```

```text
pull_request_target
+
checkout and execution of untrusted PR code
```

## Test Area 10: Remote Script Execution

### Negative Check

Flag patterns such as:

```bash
curl https://example.com/install.sh | sh
wget https://example.com/install.sh -O- | bash
```

### Manual Verification

If this pattern is used, verify:

- why it is needed,
- whether the version is pinned,
- whether checksum/signature verification exists,
- whether the job has secrets,
- whether a safer installation method exists.

## Test Area 11: Docker Base Image Review

### Positive Check

Docker base images should be reviewed and updated intentionally.

### Stronger Control

Pin base images by digest where appropriate:

```dockerfile
FROM node@sha256:<digest>
```

### Additional Checks

- image scanning runs,
- final image is minimal,
- final image runs as non-root where possible,
- SBOM is generated for the image,
- images are signed or attested where appropriate.

## Test Area 12: Release Artifact Integrity

### Positive Check

Verify that:

- tests run before packaging,
- the packaged artifact is the artifact tested or traceable to the tested commit,
- SBOM is generated for the release artifact,
- release process is restricted to trusted branches,
- deployment secrets are not available to untrusted workflows.

## Manual Verification Checklist

Before closing an A03 supply chain review, verify:

- lockfiles exist and are committed,
- CI uses `npm ci` where possible,
- lifecycle scripts are reviewed,
- `--ignore-scripts` is used where appropriate,
- dependency scanning exists,
- SBOM is generated,
- GitHub Actions are pinned or reviewed,
- workflow permissions are minimal,
- secrets are scoped carefully,
- remote script execution is avoided or verified,
- release artifacts are traceable.

## What Should Fail After the Fix

After remediation, these should fail or be flagged:

```text
CI install without lockfile.
CI package job using dynamic dependency resolution without justification.
New postinstall script without review.
Remote curl | sh script without verification.
Third-party GitHub Action added without pinning or review.
Release without SBOM generation where SBOM is required.
Dependency scan with untriaged critical finding.
```

## What Should Continue to Work After the Fix

These should still work:

```text
Local development install flow.
CI lint/test/build workflows.
Frontend and backend builds.
SBOM generation.
Dependency update process.
Release packaging.
Docker build and smoke tests.
```

## Acceptance Criteria

A supply chain remediation can be accepted when:

- dependency installation is reproducible,
- CI uses lockfile-aware installation,
- lifecycle scripts are reviewed and controlled,
- dependency scanning is enabled,
- SBOM generation is part of release/package flow,
- actions and remote scripts are reviewed,
- secrets are scoped appropriately,
- release artifacts are traceable to source and tests.

## Summary

A03 regression checks should ensure that dependency and build controls remain stable over time.

The main goal is:

> make dependency installation, CI/CD execution, and release artifacts predictable, reviewable, and auditable.
