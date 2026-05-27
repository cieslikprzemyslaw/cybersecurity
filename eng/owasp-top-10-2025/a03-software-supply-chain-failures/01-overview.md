# A03:2025 - Software Supply Chain Failures

## What This OWASP Category Means

Software Supply Chain Failures happen when an application is weakened through the external code, tools, build process, CI/CD pipeline, dependencies, or release artifacts it relies on.

In a Node.js or frontend project, the software supply chain includes more than the source code written by the development team. It also includes:

- `package.json`,
- direct dependencies,
- transitive dependencies,
- npm registry packages,
- npm lifecycle scripts,
- package manager behaviour,
- lockfiles,
- CI/CD workflows,
- GitHub Actions,
- Docker images,
- build tools,
- release artifacts,
- SBOM and dependency inventory,
- dependency update tools,
- secrets used during builds and deployments.

In simple terms:

> A supply chain issue means the application can become risky because something it trusts during install, build, test, package, or deploy is compromised, outdated, uncontrolled, or not reproducible.

## Why It Matters

Modern frontend and Node.js applications usually depend on many third-party packages and automated pipeline steps.

This creates a large trust chain:

```text
source code
  -> package manager
  -> dependencies
  -> transitive dependencies
  -> install scripts
  -> CI/CD workflows
  -> build tools
  -> Docker images
  -> release artifacts
  -> production deployment
```

A weakness at any point can affect the final application.

Examples:

- a malicious npm package runs code during installation,
- a compromised dependency is pulled during CI,
- a build uses different dependency versions from one day to another,
- a GitHub Action tag changes upstream,
- a Docker base image changes unexpectedly,
- a CI job downloads and runs a remote script without integrity checks,
- a secret is available to a job that executes untrusted code,
- a release artifact cannot be traced back to the tested source code.

## Practical Review Context

For this category, I reviewed a public open-source Node/JavaScript project from a supply chain perspective.

The goal was not to report issues against the project. The goal was to learn how an AppSec engineer reviews common supply chain controls and risks in a real-looking codebase.

The review covered:

- root `package.json`,
- npm scripts,
- npm lifecycle scripts,
- `postinstall`,
- `npm install --ignore-scripts`,
- dependency categories,
- SBOM generation,
- GitHub Actions workflows,
- CI/CD install behaviour,
- lockfile/reproducibility questions,
- remote script execution patterns,
- GitHub Actions pinning.

## Common Examples

Software Supply Chain Failures can include:

- missing or ignored lockfiles,
- CI using dynamic dependency resolution,
- vulnerable dependencies,
- abandoned dependencies,
- malicious packages,
- typosquatting packages,
- dependency confusion,
- npm lifecycle scripts running unexpectedly,
- `preinstall`, `install`, `postinstall`, or `prepare` scripts executing code,
- CI jobs running `npm install` without lockfile controls,
- remote install scripts such as `curl ... | sh`,
- GitHub Actions referenced only by mutable tags,
- Docker base images not pinned by digest,
- missing dependency scanning,
- missing SBOM,
- missing artifact provenance,
- unclear release process,
- secrets exposed to jobs that execute untrusted code.

## Common Root Causes

Typical root causes include:

- lack of dependency ownership,
- no lockfile or reproducible build strategy,
- dependency ranges that resolve differently over time,
- overtrusting npm lifecycle scripts,
- no review of CI/CD install steps,
- no dependency scanning or alerting,
- no SBOM or component inventory,
- no process for dependency updates,
- mutable third-party CI/CD actions,
- build jobs with excessive permissions,
- secrets available in risky pipeline contexts,
- release artifacts not tied to reviewed source and tests,
- convenience-focused build scripts that hide failures or execute too much automatically.

## Impact

The impact depends on which part of the supply chain is affected.

Possible impacts include:

- malicious code execution during install or build,
- compromised CI/CD runner,
- stolen CI/CD secrets,
- vulnerable code included in production,
- unexpected dependency changes,
- non-reproducible builds,
- vulnerable release artifacts,
- reduced incident response visibility,
- inability to quickly answer “are we using this vulnerable package?”,
- deployment of an artifact that was not the same as the tested artifact.

## Key Concepts I Practised

### npm Lifecycle Scripts

npm lifecycle scripts such as `preinstall`, `install`, `postinstall`, and `prepare` can execute automatically during package installation.

This is not always bad. They can be used legitimately for setup, native builds, or generated files.

However, from a supply chain perspective, they are important because they can execute code automatically in a developer machine or CI environment.

### `npm install --ignore-scripts`

During the review, I saw `npm install --ignore-scripts` used in CI.

I had not used this as an everyday frontend pattern before. I checked what it does and learned that it installs dependencies but prevents npm lifecycle scripts from running.

This can be a useful security control when a CI job only needs dependencies installed and does not need package scripts to execute.

It does not remove all supply chain risk, but it reduces one common risk: automatic code execution during install.

### Lockfiles and Reproducibility

A lockfile such as `package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml` records the exact dependency versions used by the project.

Without a committed lockfile, future installs can resolve dependency ranges differently. This means a build today and a build next month may not install the exact same transitive dependency versions.

This matters for security because a new dependency version can enter the build without a source code change.

### `npm install` vs `npm ci`

`npm install` is common for local development. It installs dependencies and can update the lockfile if one exists.

`npm ci` is designed for CI environments. It installs exactly from the lockfile and fails if `package.json` and the lockfile are out of sync.

For many CI/CD pipelines, `npm ci` plus a committed lockfile is a stronger reproducibility control than `npm install`.

### SBOM

SBOM means Software Bill of Materials.

During this review, I saw SBOM generation with CycloneDX in a practical build/release context. This was not something I had seen often in everyday frontend `package.json` files.

An SBOM does not fix vulnerabilities by itself. It creates an inventory of components included in the application or artifact.

This helps with:

- vulnerability management,
- audits,
- incident response,
- dependency visibility,
- answering whether a project uses a known vulnerable component.

### GitHub Actions Pinning

Using a GitHub Action by a mutable tag such as `@v4` is convenient, but the tag can point to different code over time.

Pinning an action to a commit SHA improves reproducibility and reduces supply chain risk because the workflow uses a specific version of the action.

### Remote Script Execution

A pattern such as:

```bash
curl https://example.com/install.sh | sh
```

is a supply chain review point because it downloads a remote script and immediately executes it.

This may be acceptable in some contexts, but it should be reviewed carefully, especially in CI jobs that have access to secrets.

Safer approaches can include:

- pinned versions,
- checksum verification,
- signature verification,
- trusted package manager installation,
- limiting secrets and permissions in that job.

## Related Vulnerabilities I Have Already Practised

Related topics from my previous learning:

- [HTTP, Request/Response and Auth Basics](../../fundamentals/01-http-request-response-auth.md)
- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md)
- [Security Misconfiguration](../a02-security-misconfiguration/01-overview.md)
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)

## My Takeaway as a Frontend Engineer

Frontend projects can have a large supply chain because they often rely on many npm packages, build tools, bundlers, test tools, CI scripts, and deployment workflows.

A project can have safe application code but still carry risk through:

- dependency installation,
- package scripts,
- transitive dependencies,
- CI/CD workflow configuration,
- release artifact creation,
- secrets available during builds.

The main mindset shift is:

> AppSec review is not only about request/response vulnerabilities. It also includes how the application is built, what it trusts, and whether the final artifact is reproducible and traceable.

## What Good Looks Like

A safer supply chain setup should include:

- committed lockfiles,
- `npm ci` in CI where appropriate,
- careful use of lifecycle scripts,
- `--ignore-scripts` where lifecycle scripts are not needed,
- dependency scanning,
- secret scanning,
- dependency update automation,
- SBOM generation,
- reviewed CI/CD permissions,
- GitHub Actions pinned to specific versions or SHA,
- minimal secrets in build jobs,
- trusted installation methods,
- clear release artifact process,
- artifact provenance or attestations where possible,
- documented ownership of dependency risk.

## Summary

A03 is about trust in the software delivery chain.

The most important things I learned from this review were:

1. `postinstall` and other lifecycle scripts are important supply chain review points.
2. `--ignore-scripts` can reduce install-time script execution risk when used intentionally.
3. Lockfiles and `npm ci` help make builds more reproducible.
4. SBOM gives visibility into components, but does not replace scanning or remediation.
5. `curl | sh` in CI should be reviewed carefully.
6. Pinning GitHub Actions to SHA improves reproducibility and reduces third-party action risk.
