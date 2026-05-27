# A03:2025 - Software Supply Chain Failures Checklist

## Purpose

Use this checklist when reviewing a frontend, Node.js, or full-stack JavaScript project for software supply chain risks.

The goal is to understand what the application trusts during install, build, test, package, and deployment.

## Core Question

Ask:

> Could something outside our source code change what gets installed, built, tested, packaged, or deployed?

## 1. package.json Review

### Questions

- What scripts run during install, build, test, package, and deploy?
- Are there lifecycle scripts such as `preinstall`, `install`, `postinstall`, or `prepare`?
- Do scripts execute remote code?
- Do scripts download files from the internet?
- Do scripts ignore build failures?
- Do scripts run nested installs in subdirectories?
- Do scripts access secrets or environment variables?
- Are scripts understandable and documented?

### Red Flags

- unexpected `postinstall`,
- `curl ... | sh`,
- `wget ... | bash`,
- dynamic downloads during build,
- scripts that hide failures with `|| true` or similar,
- scripts that modify source unexpectedly,
- scripts that run in CI with deployment secrets available,
- nested `npm install` without clear reason.

## 2. npm Lifecycle Scripts

### Scripts to Review

```text
preinstall
install
postinstall
prepare
prepublish
prepack
postpack
```

### Questions

- Does this script need to run in CI?
- Does it run automatically during install?
- Does it execute third-party code?
- Does it compile native modules?
- Does it download anything?
- Does it need access to secrets?
- Can it be disabled in some jobs with `--ignore-scripts`?

### Safe Implementation Notes

Lifecycle scripts are not always bad, but they should be intentional.

Good practice:

- document why the script exists,
- keep it minimal,
- avoid network downloads where possible,
- avoid secrets in jobs that run lifecycle scripts,
- use `--ignore-scripts` in jobs that do not need scripts,
- review lifecycle scripts in dependencies when risk is high.

## 3. Dependency Review

### Questions

- Which dependencies are direct?
- Which dependencies are transitive?
- Which dependencies are security-sensitive?
- Are any dependencies outdated or abandoned?
- Are any dependencies known to be vulnerable?
- Are there packages with suspicious names?
- Are there packages that should be devDependencies instead of dependencies?
- Are dependencies maintained?
- Are dependency updates automated?

### Security-Sensitive Dependency Areas

Pay extra attention to packages that handle:

- authentication,
- JWTs,
- sessions,
- cookies,
- sanitisation,
- file uploads,
- archive extraction,
- XML/YAML parsing,
- template rendering,
- CORS,
- security headers,
- error handling,
- native/binary installation,
- database access,
- HTTP clients.

## 4. Lockfile and Reproducibility

### Questions

- Is a lockfile committed?
- Which package manager is used?
- Is `package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml` present?
- Does CI use `npm ci` where possible?
- Does CI use `npm install` without a lockfile?
- Are dependency versions reproducible?
- Can the same commit build the same dependency tree later?
- Are package manager settings disabling lockfile generation?

### Red Flags

- no lockfile,
- lockfile ignored in `.gitignore`,
- `package-lock=false`,
- CI uses repeated `npm install`,
- dependencies use broad ranges without lockfile control,
- production builds depend on live dependency resolution.

### Safer Approach

Where appropriate:

- commit the lockfile,
- use `npm ci` in CI,
- review lockfile changes in pull requests,
- avoid accidental lockfile updates,
- scan lockfile dependencies,
- document package manager expectations.

## 5. CI/CD Dependency Installation

### Questions

- Which jobs install dependencies?
- Do they use `npm install`, `npm ci`, or another package manager?
- Do install scripts run?
- Is `--ignore-scripts` used where scripts are not needed?
- Do jobs with secrets also run dependency scripts?
- Are dependencies installed before or after secrets are made available?
- Are global packages installed during CI?
- Are global installs pinned?

### Red Flags

- normal `npm install` in sensitive jobs,
- dependency scripts running with secrets available,
- global installs without pinned versions,
- install commands downloading from untrusted locations,
- remote scripts executed directly.

## 6. `--ignore-scripts` Review

### Why It Matters

`npm install --ignore-scripts` installs packages without running lifecycle scripts.

This can reduce risk in jobs where lifecycle scripts are not required.

### Questions

- Is it used in lint or static analysis jobs?
- Are there jobs where it could be safely used?
- Are there jobs where lifecycle scripts are genuinely required?
- Is the difference documented?

### Important Note

`--ignore-scripts` is a control, not a full solution.

It only helps in the jobs where it is used.

## 7. SBOM Review

### Questions

- Is an SBOM generated?
- Is it generated locally only or in CI/release?
- Is the SBOM included with artifacts?
- Is it in a standard format such as CycloneDX or SPDX?
- Is it scanned against vulnerability databases?
- Is it used during incident response?
- Is it generated for production dependencies only or all dependencies?

### Good Practice

SBOM is useful when it is:

- generated automatically,
- attached to release artifacts,
- kept up to date,
- used by scanning or vulnerability management tools,
- reviewed during dependency incidents.

### Important Note

SBOM provides visibility. It does not fix vulnerabilities by itself.

## 8. Dependency Scanning and Updates

### Questions

- Is Dependabot, Renovate, Snyk, OSV Scanner, Trivy, Grype, or another scanner used?
- Are alerts enabled?
- Are updates grouped or reviewed?
- Is there a process for high/critical dependency issues?
- Are devDependency findings triaged separately from runtime dependency findings?
- Are false positives documented?
- Are security updates tested before merge?

### Red Flags

- no dependency scanning,
- alerts ignored,
- no owner for dependency updates,
- no policy for critical vulnerabilities,
- old legacy update configuration that may no longer be active,
- no CI gate or manual review process.

## 9. GitHub Actions Review

### Questions

- Are actions pinned to commit SHA?
- Are actions only pinned to mutable tags such as `@v4`?
- Are third-party actions trusted?
- Are permissions restricted?
- Are secrets exposed to pull request workflows?
- Are risky events like `pull_request_target` handled safely?
- Do workflows checkout and execute untrusted PR code?
- Are publish/deploy jobs restricted to trusted branches?

### Positive Controls

- SHA-pinned actions,
- minimal permissions,
- restricted deploy jobs,
- no secrets in untrusted PR jobs,
- gated publish steps,
- reviewed third-party actions.

### Red Flags

- third-party actions pinned only by tag,
- broad permissions,
- secrets exposed to untrusted code,
- `pull_request_target` with checkout of PR code,
- deployment from forks,
- auto-commit/push workflows with unclear permissions.

## 10. Remote Script Execution

### Questions

- Does CI run `curl | sh` or `wget | bash`?
- Is the installer version pinned?
- Is checksum verification used?
- Is signature verification used?
- Does the job have secrets?
- Can the tool be installed through a trusted package manager instead?
- Is the remote endpoint trusted and stable?

### Red Flags

```bash
curl https://example.com/install.sh | sh
wget https://example.com/install.sh -O- | bash
```

### Safer Options

- pinned installer version,
- checksum verification,
- signature verification,
- trusted package manager,
- prebuilt trusted image,
- minimal secrets in that job.

## 11. Docker and Artifact Review

### Questions

- Are Docker base images pinned by digest or only tag?
- Are images updated intentionally?
- Is the final image minimal?
- Does the final image run as non-root?
- Is an SBOM generated for the image?
- Is the image scanned?
- Is the image signed?
- Is provenance/attestation generated?
- Is the artifact that passed tests the one that is deployed?

## 12. Secrets in CI/CD

### Questions

- Which jobs have access to secrets?
- Do those jobs run dependency install scripts?
- Do those jobs run untrusted code?
- Are secrets available on pull request workflows?
- Are secrets scoped only to required jobs?
- Are secrets rotated if exposed?
- Is secret scanning enabled?

### Red Flags

- deployment tokens available too early,
- secrets in jobs that run untrusted dependency scripts,
- secrets in pull request workflows from forks,
- secrets echoed to logs,
- broad repository tokens.

## 13. Remediation Checklist

For missing lockfile/reproducibility:

- add and commit a lockfile,
- use `npm ci` in CI,
- review lockfile changes,
- document package manager policy,
- scan lockfile dependencies.

For lifecycle script risk:

- review lifecycle scripts,
- document why they are needed,
- use `--ignore-scripts` where possible,
- avoid secrets in jobs that run install scripts,
- reduce network access during install where possible.

For remote script execution:

- avoid `curl | sh` where possible,
- pin installer versions,
- verify checksums or signatures,
- use trusted package managers,
- reduce job permissions and secrets.

For GitHub Actions risk:

- pin actions to commit SHA,
- restrict workflow permissions,
- review third-party actions,
- avoid secrets in untrusted contexts,
- restrict publish/deploy jobs to trusted branches.

For dependency visibility:

- enable dependency scanning,
- enable secret scanning,
- generate SBOM,
- attach SBOM to artifacts,
- define ownership for dependency updates.

## Frontend Engineer Takeaway

A frontend project can have supply chain risk even when the application code looks safe.

For AppSec review, I should look beyond React/TypeScript code and also review:

- package scripts,
- dependency installation,
- lockfiles,
- CI/CD workflows,
- GitHub Actions,
- Docker images,
- build artifacts,
- dependency scanning,
- SBOM,
- secrets in the pipeline.

This is a different type of security thinking than XSS, IDOR, or SQL Injection, but it is very relevant for modern frontend and Node.js projects.
