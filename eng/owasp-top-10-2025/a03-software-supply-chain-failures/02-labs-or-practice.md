# A03:2025 - Software Supply Chain Failures: Labs and Practice

## Purpose

This file documents the practical work completed for A03:2025 - Software Supply Chain Failures.

The goal was to practise supply chain thinking using a public open-source Node/JavaScript project, not to report issues against that project.

This category is more process/build/dependency-focused than request/response-focused, so the practice was based on reviewing `package.json`, npm scripts, dependency installation, CI/CD workflows, and build/release controls.

## Completed Practice

### OWASP Juice Shop GitHub Repository Review

**Target:** OWASP Juice Shop
**Source:** GitHub repository review
**Target type:** Public open-source Node/JavaScript project
**Focus:** Package and CI/CD supply chain review  
**Category:** A03:2025 - Software Supply Chain Failures  
**Status:** Completed after review and quiz
**Main pattern:** Supply chain review of npm lifecycle scripts, dependency reproducibility, SBOM visibility, and GitHub Actions controls

External link:

- https://github.com/juice-shop/juice-shop

Reviewed areas:

- root `package.json`,
- npm scripts,
- dependency categories,
- npm lifecycle scripts,
- `postinstall`,
- SBOM generation,
- CI/CD workflow snippets,
- `npm install` vs `npm ci`,
- `--ignore-scripts`,
- GitHub Actions pinning,
- remote script execution pattern,
- lockfile/reproducibility concerns.

## Practice 1: package.json Review

### What I Practised

I reviewed the root `package.json` for supply chain review points:

- `scripts`,
- `dependencies`,
- `devDependencies`,
- lifecycle scripts,
- security-sensitive packages,
- build and packaging scripts,
- SBOM generation.

The main review point was the `postinstall` script:

```json
"postinstall": "cd frontend && npm install && cd .. && npm run build:frontend && (npm run --silent build:server || cd .)"
```

This was not automatically a vulnerability, but it was worth review because npm lifecycle scripts can run automatically during installation.

I also marked dependency areas that deserve extra attention because they touch security-sensitive behaviour:

- JWT/authentication,
- HTML sanitisation,
- file upload,
- archive extraction,
- XML/YAML parsing,
- CORS,
- error handling,
- directory listing,
- native/binary installation,
- security headers.

Example package areas:

```text
jsonwebtoken / express-jwt     -> token signing and verification
sanitize-html                  -> XSS and HTML sanitisation
multer                         -> file upload handling
unzipper                       -> archive extraction
libxmljs2                      -> XML parsing
js-yaml                        -> YAML parsing
cors                           -> CORS configuration
serve-index                    -> directory listing risk if misused
errorhandler                   -> development-style error handling
helmet                         -> security headers
node-pre-gyp / sqlite3         -> native/binary install surface
```

## Practice 2: CI/CD Workflow Review

### What I Practised

I reviewed GitHub Actions workflow snippets and looked for supply chain controls and risks across:

- Node.js setup,
- dependency installation,
- linting,
- unit tests,
- API tests,
- E2E tests,
- packaging,
- Docker build and push,
- Heroku deployment,
- Slack notification,
- SBOM-related build configuration.

### Main Observations

The lint job used `--ignore-scripts`:

```bash
npm install --ignore-scripts
cd frontend
npm install --ignore-scripts
```

This reduces install-time lifecycle script execution in jobs that do not need those scripts.

Many GitHub Actions were pinned to commit SHA values rather than only moving tags:

```yaml
uses: actions/checkout@<commit-sha> # version comment
```

This improves reproducibility and reduces trust in mutable upstream tags.

The workflow also contained a remote install pattern:

```bash
curl https://cli-assets.heroku.com/install.sh | sh
```

This is a review point because the pipeline downloads a remote script and executes it, especially if the job has deployment secrets.

The strongest concern was dependency reproducibility:

- no normal committed root or frontend lockfile was observed in active use,
- several jobs used normal `npm install`,
- dependency versions may resolve differently over time.

Positive controls also existed:

- some jobs used `npm install --ignore-scripts`,
- many GitHub Actions were pinned to SHA,
- SBOM generation was present through CycloneDX,
- the workflow had structured test/build/package stages.

## Key Takeaways

The most important A03 learning points from this practice were:

1. `postinstall` and lifecycle scripts are important supply chain review points.
2. `--ignore-scripts` can reduce install-time code execution risk when used intentionally.
3. Missing lockfiles can make builds less reproducible.
4. `npm ci` is usually stronger than `npm install` in CI when a lockfile exists.
5. SBOM gives useful dependency visibility but does not fix vulnerabilities by itself.
6. `curl | sh` should be reviewed carefully in CI/CD.
7. SHA-pinned GitHub Actions improve reproducibility and reduce third-party action risk.

Detailed review guidance is kept in:

- [A03 overview](01-overview.md)
- [A03 learning notes](05-learning-notes.md)
- [A03 checklist](03-checklist.md)
- [A03 regression tests](04-regression-tests.md)
- [Example supply chain finding](security-findings/01-example-finding.md)

## Related Notes

- [HTTP, Request/Response and Auth Basics](../../fundamentals/01-http-request-response-auth.md)
- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md)
- [Security Misconfiguration](../a02-security-misconfiguration/01-overview.md)
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)
