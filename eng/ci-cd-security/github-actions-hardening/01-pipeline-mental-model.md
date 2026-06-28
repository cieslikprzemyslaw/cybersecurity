# CI/CD Security Mental Model

## What CI/CD security means

CI/CD security protects the path between a code change and a trusted build or merge.

A pipeline is not only a helper. It is a trusted system that can execute code, install packages, run scripts, build containers and sometimes access tokens or secrets.

That means a weak pipeline can become a way to bypass normal application security controls.

## What can go wrong

A CI/CD pipeline can be abused or misconfigured in several ways:

```text
workflow token has too many permissions
untrusted code runs with access to secrets
malicious dependency script runs during install
container builds but fails at runtime
known vulnerable dependency is ignored
SAST findings are never reviewed
branch protection can be bypassed
logs expose sensitive values
duplicate workflow runs create noise and hide real failures
```

## Practical model

The sprint used this layered model:

```text
1. Validate code
   format, lint, typecheck, tests, build, Prisma validation

2. Validate runtime
   Docker build, Docker run, API health check, logs on failure

3. Validate dependencies
   npm audit JSON and a policy script

4. Review source-code patterns
   CodeQL for JavaScript/TypeScript

5. Protect the repository flow
   no direct push to master, pull request based changes, required checks

6. Keep the pipeline clean
   less duplicate CI noise, ignored local files, no unnecessary secrets
```

## Why least privilege matters

A normal validation job usually needs to read the repository. It should not receive write permissions without a reason.

Safer default:

```yaml
permissions:
  contents: read
```

This reduces the blast radius if something inside the job behaves unexpectedly.

## Why timeout and concurrency matter

Timeouts stop jobs from running forever.

Concurrency cancels old runs when a newer commit arrives.

These settings improve reliability, reduce waste and make pull request review easier to understand.

## Why runtime validation matters

A successful build does not prove that the application starts correctly.

For a Docker-based API, the pipeline should also check:

```text
the image builds
the container starts
required environment variables are present
the API listens on the expected port
the health endpoint responds
logs are available when something fails
cleanup runs even on failure
```

## Why scanners still need triage

Dependency scanners and SAST tools are useful, but they do not replace engineering judgement.

Good triage asks:

```text
Is this dependency direct or transitive?
Is a fix available?
Is this code used at runtime or only during development?
Can the issue be exploited in this project?
Should we fix now or document temporary risk acceptance?
When does the accepted risk expire?
```

## Final mental model

A good CI/CD security setup should make the safe path easy and the risky path visible.

The goal is not to add tools for the sake of tools. The goal is to create useful checks that give confidence before merge.
