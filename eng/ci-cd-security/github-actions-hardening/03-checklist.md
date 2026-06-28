# CI/CD Security Checklist

Use this checklist when reviewing a GitHub Actions based project.

## Workflow permissions

- [ ] Does the workflow define explicit permissions?
- [ ] Do validation jobs use read-only access when possible?
- [ ] Are write permissions limited to jobs that really need them?
- [ ] Are deployment permissions separated from pull request validation?

Good baseline:

```yaml
permissions:
  contents: read
```

## Reliability controls

- [ ] Does each job have a timeout?
- [ ] Does the workflow use concurrency to cancel old runs?
- [ ] Are failure logs available for runtime checks?
- [ ] Does cleanup run with `if: always()` where needed?

## Pull request validation

- [ ] Does CI run on pull requests to the protected branch?
- [ ] Does CI run on push to the protected branch after merge?
- [ ] Are duplicate PR branch push runs avoided?
- [ ] Are required checks easy to understand?

## Build and runtime validation

- [ ] Does the project run format/lint/typecheck/tests/build?
- [ ] Does the database or schema layer have explicit validation?
- [ ] If Docker is used, does CI build the image?
- [ ] Does CI run the container?
- [ ] Does CI check a health endpoint?
- [ ] Are runtime environment variables explicit?

## Dependency security

- [ ] Is dependency scanning enabled?
- [ ] Are high/critical vulnerabilities blocked?
- [ ] Is there a policy for temporary risk acceptance?
- [ ] Does accepted risk require a reason?
- [ ] Does accepted risk require an expiry date?
- [ ] Does accepted risk require a follow-up issue?
- [ ] Are direct dependency fixes preferred before overrides?

## SAST

- [ ] Is static analysis enabled for the language stack?
- [ ] Are findings reviewed instead of ignored?
- [ ] Is the first phase focused on visibility and triage?
- [ ] Is there a plan for confirmed findings?
- [ ] Are false positives documented carefully?

## Repository protection

- [ ] Is direct push to the protected branch blocked?
- [ ] Are pull requests required before merge?
- [ ] Are status checks required before merge?
- [ ] Can administrators bypass protections?
- [ ] Is the bypass setting a conscious decision?

## Secrets hygiene

- [ ] Is `.env` ignored?
- [ ] Are local database files ignored?
- [ ] Are logs ignored?
- [ ] Are uploads/runtime data ignored?
- [ ] Are local Docker secrets ignored?
- [ ] Are real secrets absent from the repository?
- [ ] Are workflows avoiding unnecessary secrets?
- [ ] Are secrets not printed to logs?

## Final review question

```text
If a risky change entered this repository, which control would catch it?
```
