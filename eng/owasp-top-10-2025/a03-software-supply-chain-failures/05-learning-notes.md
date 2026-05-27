# A03:2025 - Software Supply Chain Failures Learning Notes

These notes preserve the longer learning reflections from the A03 GitHub repository review. The shorter practice evidence stays in [02-labs-or-practice.md](02-labs-or-practice.md).

## What I Learned About `postinstall`

`postinstall` runs automatically after `npm install`.

Before this A03 review, I knew that npm scripts existed, but I had not looked at `postinstall` from a supply chain perspective in detail.

I learned that this kind of script is important because it can execute commands automatically during installation.

In the reviewed project, the script:

1. enters the `frontend` directory,
2. runs another `npm install`,
3. goes back to the root directory,
4. builds the frontend,
5. attempts to build the server.

This can be legitimate in a complex project. It can help prepare nested applications automatically.

However, from a supply chain perspective, it is a review point because lifecycle scripts can execute code during installation. If a package or dependency is malicious or compromised, install-time scripts are a common place where code execution can happen.

Important nuance:

> A `postinstall` script is not automatically a vulnerability.

The right AppSec question is:

> Does this script execute only expected, reviewed commands, and is it safe to run in local development and CI/CD environments?

## `--ignore-scripts`

I saw this pattern in the lint job:

```bash
npm install --ignore-scripts
cd frontend
npm install --ignore-scripts
```

I did not initially know exactly what `--ignore-scripts` did, so I checked it.

I learned that it installs dependencies but prevents npm lifecycle scripts such as these from running:

```text
preinstall
install
postinstall
prepare
```

This can be a good security control when the job only needs dependencies installed and does not need install scripts to run.

This is not something I normally see in everyday simple frontend `package.json` projects, but it makes sense in security-aware CI/CD contexts.

Important nuance:

`--ignore-scripts` reduces install-time script execution risk in the jobs where it is used.

It does not remove the risk everywhere if other jobs still use normal `npm install`.

In the reviewed workflow, some jobs used `--ignore-scripts`, while other test/build/package jobs used normal `npm install`.

That means lifecycle script risk is reduced in some places but still exists in other parts of the pipeline.

## Reproducibility Concern: Missing Lockfile / Dynamic Installs

During the review, I observed that there did not appear to be a normal committed root `package-lock.json` or frontend `package-lock.json` in active use, while CI used repeated `npm install` steps.

This matters because without a committed lockfile, dependency resolution is less deterministic.

A build today and a build in the future may install different transitive dependency versions without any source code change.

This was one of the strongest A03 review points.

## `npm install` vs `npm ci`

I learned the difference more clearly during this review.

`npm install`:

- is common in local development,
- installs dependencies,
- can update the lockfile if one exists,
- can resolve dependency ranges dynamically.

`npm ci`:

- is designed for CI,
- requires a lockfile,
- installs exactly from the lockfile,
- fails if `package.json` and the lockfile are out of sync,
- is usually better for reproducible CI builds.

## SBOM

The project had SBOM-related configuration using CycloneDX.

I saw SBOM generation in practice for the first time during this review.

SBOM means Software Bill of Materials. It is an inventory of components included in the application or artifact.

I learned that SBOM is useful because it helps answer questions like:

- What dependencies are included in this artifact?
- Are we using a package affected by a new vulnerability?
- Which components do we need to review during an incident?
- What is inside the release package?

Important clarification:

> SBOM does not fix vulnerabilities by itself. It gives visibility. It becomes stronger when combined with dependency scanning, vulnerability databases, alerting, and remediation processes.

## GitHub Actions Pinning

Many GitHub Actions in the workflow were pinned to commit SHA values rather than only using moving tags.

This is a good supply chain control because it improves reproducibility and reduces the risk of silently pulling changed action code through mutable tags.

Example pattern:

```yaml
uses: actions/checkout@<commit-sha> # version comment
```

Some actions appeared to use tags instead of commit SHA.

This is not automatically a critical issue, but it is a review point because tags can move or change over time.

A stronger approach is to pin actions to a reviewed SHA and update them intentionally.

## `curl | sh`

The workflow contained a remote install pattern similar to:

```bash
curl https://cli-assets.heroku.com/install.sh | sh
```

This is a classic CI/CD supply chain review point.

The risk is that the pipeline downloads a remote script and immediately executes it.

This should be reviewed especially when the job has access to deployment secrets.

Better controls can include:

- pinned installer versions,
- checksum verification,
- signature verification,
- trusted package manager installation,
- limiting job permissions,
- limiting secrets available to the job.

## Supply Chain Review Is Not Only About Vulnerable Packages

Before this, I mostly associated dependency security with known vulnerable packages.

This review showed me that A03 also includes:

- install scripts,
- build reproducibility,
- CI/CD workflow safety,
- GitHub Actions pinning,
- SBOM,
- release artifacts,
- remote installer scripts,
- secrets in pipeline contexts.

## Some Security Controls Are Not Everyday Frontend Patterns

I do not normally see things like SBOM generation or `--ignore-scripts` in small everyday frontend projects.

This review helped me understand when they make sense:

- `--ignore-scripts` makes sense when a CI job does not need lifecycle scripts to run.
- SBOM makes sense when a project wants visibility into the components included in an artifact.
- SHA-pinning GitHub Actions makes sense when a project wants stronger CI/CD reproducibility.

## Biggest Review Point

The strongest concern was not one specific package.

The strongest concern was the combination of:

- no active committed lockfile observed,
- repeated `npm install` usage,
- dependency ranges that may resolve differently over time.

This can make builds less reproducible and increase exposure to unexpected dependency changes.

## Good Controls Can Exist Beside Risks

A review should not only list problems.

In this project, I saw positive controls such as:

- SBOM generation,
- some use of `--ignore-scripts`,
- many GitHub Actions pinned to SHA,
- gated Docker publishing,
- structured test/build workflows.

At the same time, there were still review points such as:

- normal `npm install` in several jobs,
- missing lockfile/reproducibility concerns,
- `curl | sh`,
- some actions referenced by tag.

## Real-World Review Angle

In a real project, I would use this A03 review pattern to check:

- Is there a committed lockfile?
- Does CI use `npm ci` where possible?
- Are lifecycle scripts needed in CI?
- Can any job use `--ignore-scripts`?
- Are GitHub Actions pinned to SHA or only tags?
- Are third-party actions trusted and reviewed?
- Are secrets available to jobs that run dependency scripts?
- Is dependency scanning enabled?
- Is secret scanning enabled?
- Is SBOM generated for release artifacts?
- Are Docker base images pinned or reviewed?
- Are release artifacts tied to tested source code?
- Is `curl | sh` avoided or verified?

## Summary

The most important A03 learning points from this practice were:

1. `postinstall` and lifecycle scripts are important supply chain review points.
2. `--ignore-scripts` can reduce install-time code execution risk when used intentionally.
3. Missing lockfiles can make builds less reproducible.
4. `npm ci` is usually stronger than `npm install` in CI when a lockfile exists.
5. SBOM gives useful dependency visibility but does not fix vulnerabilities by itself.
6. `curl | sh` should be reviewed carefully in CI/CD.
7. SHA-pinned GitHub Actions improve reproducibility and reduce third-party action risk.
