# Evidence

## Merged PRs

The CI/CD Security sprint was implemented through small merged pull requests.

| PR | Title | Evidence value |
|---|---|---|
| #365 | chore: harden CI validation workflow | Added CI hardening controls such as least-privilege permissions, timeout, concurrency and explicit validation steps. |
| #366 | chore: validate Docker runtime in CI | Added Docker image build, container startup, health check, logs on failure and cleanup. |
| #367 | chore: add dependency audit policy | Added npm audit JSON policy and high/critical vulnerability gate with documented risk acceptance. |
| #368 | chore: add CodeQL SAST workflow | Added JavaScript/TypeScript SAST visibility through CodeQL. |
| #369 | chore: avoid duplicate CI runs for PR branches | Cleaned CI triggers to reduce duplicate PR branch runs and improve pipeline signal. |

Supporting context:

| PR | Title | Evidence value |
|---|---|---|
| #364 | Chore/docker api image | Provided Docker image context used later by Docker runtime validation. |

## Final CI screenshot

Screenshot file:

[ci-cd-security-github-actions-success.png](../../../assets/ci-cd-security-github-actions-success.png)

The screenshot shows:

```text
Workflow: ci.yml
Trigger: push to master
Status: Success
Total duration: 3m 59s
Validate: 2m 57s
Docker Runtime: 54s
Dependency Security: 25s
Vitest: 138 test files, 545 tests, all passed
```

## What this proves

The screenshot proves that the final merged pipeline successfully ran the main validation layers after the sprint changes:

```text
Validate
Docker Runtime
Dependency Security
```

The PR list proves that CodeQL and trigger cleanup were also added as separate reviewed changes.

## Repository protection evidence

Repository protection was confirmed by testing that direct push to `master` is blocked, even for the owner.

That means changes must follow:

```text
branch -> pull request -> checks -> merge
```

## Secrets hygiene evidence

Local review confirmed:

```text
.env ignored
local database files ignored
logs ignored
uploads ignored
local Docker secrets ignored
workflow does not require secrets
local secret-like scan passed
```

## Final evidence statement

```text
The AppSec Report Builder project now has a layered CI/CD security baseline with code validation, Docker runtime validation, dependency vulnerability policy, CodeQL SAST, protected branch flow, cleaner triggers and repository secret hygiene.
```
