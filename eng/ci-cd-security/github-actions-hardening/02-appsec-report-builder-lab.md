# Practical Lab: AppSec Report Builder CI/CD Security

## Goal

The goal was to improve the AppSec Report Builder pipeline in small, reviewable steps.

The project already had a working CI workflow. The sprint added security and reliability layers around it.

## Starting point

The existing workflow already did useful validation:

```text
npm ci
format check
lint
typecheck
tests
build
```

This was a good base, but it did not fully answer security questions such as:

```text
Does the workflow use minimal permissions?
Does the Docker image actually start?
Are high or critical dependency vulnerabilities blocked?
Is SAST available for JavaScript/TypeScript?
Can master be changed without review?
Are secrets protected from accidental commits?
Are duplicate CI runs creating noise?
```

---

## Step 1: CI validation hardening

PR:

```text
#365 chore: harden CI validation workflow
```

Changes included:

```text
permissions: contents: read
timeout-minutes: 15
concurrency with cancel-in-progress
explicit Prisma validation
explicit Prisma generation
```

### Why this mattered

The job did not need write access, so `contents: read` was enough.

Timeout and concurrency made the workflow more reliable and less noisy.

Prisma checks made schema validation explicit instead of relying only on later build or test failures.

### Result

```text
CI/CD Security #1 — CI validation hardening: PASS
```

---

## Step 2: Docker runtime validation

PR:

```text
#366 chore: validate Docker runtime in CI
```

Supporting Docker image context:

```text
#364 Chore/docker api image
```

The new job built the API Docker image and started a real container in CI.

It then checked:

```text
/api/health
```

The job also printed Docker logs on failure and cleaned up the container.

### Real issue found

The first runtime check failed because Prisma expected `DATABASE_URL` before creating the Prisma client.

The fix was to run the container with:

```text
DATABASE_URL=file:/data/appsec-report-builder.db
```

### Why this mattered

This was a real CI/CD security and reliability lesson.

A container image can build correctly but still fail at runtime because required configuration is missing.

### Result

```text
CI/CD Security #2 — Docker runtime validation: PASS
```

---

## Step 3: Dependency audit policy

PR:

```text
#367 chore: add dependency audit policy
```

Instead of using a plain `npm audit --audit-level=high`, the project added a custom policy around `npm audit --json`.

The policy blocks high and critical vulnerabilities unless they are documented as temporary accepted risk.

A valid temporary acceptance needs:

```text
package
severity
reason
expiry date
follow-up issue
optional advisory ID
```

### Real issue found and fixed

The audit found a critical chain:

```text
concurrently -> shell-quote
```

`concurrently` was a direct dependency. `shell-quote` was transitive.

Decision:

```text
fix now
```

Reason:

```text
the direct dependency had a fix available
```

### Why this mattered

This created a practical triage flow:

```text
new high/critical issue appears
CI fails
engineer reviews the finding
fix now or document temporary accepted risk
accepted risk must expire
```

### Result

```text
CI/CD Security #3 — Dependency audit policy: PASS
```

---

## Step 4: CodeQL SAST

PR:

```text
#368 chore: add CodeQL SAST workflow
```

CodeQL was added for JavaScript/TypeScript.

The first goal was visibility and triage, not panic-blocking every finding immediately.

### Why this mattered

SAST can find risky patterns in source code, but findings need context.

For example, SAST can help with injection-like patterns or unsafe API use, but it may miss business logic problems such as IDOR or broken authorization.

### Result

```text
CI/CD Security #4 — CodeQL SAST visibility: PASS
```

---

## Step 5: CI trigger cleanup

PR:

```text
#369 chore: avoid duplicate CI runs for PR branches
```

The CI trigger was changed so that:

```text
pull requests to master run CI through pull_request
push to master runs CI after merge
normal PR branch pushes do not create duplicate push runs
```

### Why this mattered

This is pipeline hygiene.

It reduces wasted runner time, makes pull request checks easier to read and prevents duplicated status noise.

### Result

```text
Trigger cleanup: PASS
```

---

## Step 6: Repository protection

The repository already blocked direct push to `master`, even for the owner.

That means the normal path is:

```text
branch
pull request
checks
merge
```

### Why this mattered

Branch protection turns CI from a helpful tool into an enforced review gate.

It prevents accidental or intentional bypass of validation.

### Result

```text
Repository protection: PASS
```

---

## Step 7: Secrets and repository hygiene

The repository was checked for basic secret hygiene.

Confirmed:

```text
.env is ignored
local database files are ignored
logs are ignored
upload folders are ignored
local Docker secrets are ignored
workflow does not need secrets
local secret-like scan passed
```

### Why this mattered

Secrets should not live in source control.

Even a local-first project should avoid committing local databases, logs, uploads, `.env` files or Docker secrets.

### Result

```text
CI/CD Security #5 — Secrets and repository hygiene: PASS
```

---

## Final Result

Final CI evidence is documented in [04 Evidence](04-evidence.md). The important result was that the merged workflow completed the main CI/CD security layers:

```text
Validate
Docker Runtime
Dependency Security
```

The broader sprint also added:

```text
CodeQL / SAST
Repository Protection
Secrets Hygiene
Trigger cleanup
```

## Technical Summary

```text
I improved the AppSec Report Builder CI/CD pipeline by adding least-privilege workflow permissions, runtime Docker validation, a dependency audit policy, CodeQL SAST, branch protection checks, trigger cleanup and repository secret hygiene.
```
