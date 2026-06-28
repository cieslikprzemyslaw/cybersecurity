# CI/CD Security

This section contains practical CI/CD security notes from a frontend and application security perspective.

The goal is not only to make GitHub Actions green. The goal is to understand what the pipeline can do, what it can access, what can go wrong, and how CI/CD becomes part of the application security boundary.

## Modules

| Module | Scope |
|---|---|
| [GitHub Actions Hardening](github-actions-hardening/00-index.md) | GitHub Actions hardening, Docker runtime validation, dependency audit policy, CodeQL SAST, repository protection, trigger cleanup and secret hygiene. |

## Practical Context

The notes are based on real work in the AppSec Report Builder project.

The project already had a useful CI workflow. The sprint improved it into a layered CI/CD security pipeline with:

```text
Validate
Docker Runtime
Dependency Security
CodeQL / SAST
Repository Protection
Secrets and Repository Hygiene
```

## Learning Focus

- Treat GitHub Actions as a trusted execution environment.
- Reduce workflow permissions to the minimum needed by each job.
- Validate not only source code, but also container runtime behavior.
- Turn dependency audit output into a reviewable policy.
- Use SAST findings as triage input, not as a replacement for engineering review.
- Keep repository protections, triggers and secret hygiene visible.

## Main Lesson

A CI/CD pipeline is not just automation. It is a trusted system that installs dependencies, runs scripts, builds software and decides whether a change is safe enough to merge.

That makes it an AppSec topic.
