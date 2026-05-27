# Frontend AppSec Notes

Personal web security and application security notes from the perspective of a Frontend Engineer.

This repository is defensive and developer-oriented. It focuses on understanding web risks, practising in legal labs, improving secure implementation, and writing clearer remediation notes. It is not a collection of full walkthroughs, exploit guides, or lab answers.

## Start Here

| Area | Link | Best for |
|---|---|---|
| English notes | [eng/README.md](eng/README.md) | Main learning path and English topic index. |
| Polish notes | [pl/README.md](pl/README.md) | Polish version of the learning path. |
| OWASP Top 10 2025 EN | [eng/owasp-top-10-2025/README.md](eng/owasp-top-10-2025/README.md) | Mapping practical notes to OWASP categories. |
| OWASP Top 10 2025 PL | [pl/owasp-top-10-2025/README.md](pl/owasp-top-10-2025/README.md) | Polish OWASP mapping sprint. |
| Coverage Matrix EN | [eng/owasp-top-10-2025/coverage-matrix.md](eng/owasp-top-10-2025/coverage-matrix.md) | Current OWASP coverage and next actions. |
| Coverage Matrix PL | [pl/owasp-top-10-2025/coverage-matrix.md](pl/owasp-top-10-2025/coverage-matrix.md) | Polish coverage tracking. |

## What This Repository Covers

The notes are built around practical AppSec learning:

- web fundamentals: HTTP, cookies, sessions, auth, request/response flow,
- Burp Suite basics and manual request testing,
- authentication and authorization issues,
- XSS, SQL injection, CSRF, path traversal, file upload bugs and SSRF,
- API and configuration security,
- OWASP Top 10 2025 mapping,
- developer-friendly remediation and regression testing.

The goal is to connect AppSec concepts with real development work:

- What can the user control?
- Where does that input go?
- What does the backend trust?
- What happens if a request is modified?
- Is this an authentication issue or an authorization issue?
- What would a safe implementation look like?
- How could this be tested to prevent regression?

## Learning Approach

The learning model is intentionally practical:

- **10% theory** to understand the risk.
- **70% hands-on labs** to practise safely.
- **20% notes and review** to turn practice into understanding.

For each topic, the focus is root cause, impact, trust boundaries, developer mistakes, secure implementation, remediation and regression testing.

## Repository Structure

| Path | Purpose |
|---|---|
| `eng/fundamentals/` | Reusable Web AppSec foundations before vulnerability-specific study. |
| `eng/key-web-vulnerabilities/` | English topic modules for common web vulnerability classes. |
| `eng/owasp-top-10-2025/` | Practical OWASP Top 10 2025 mapping sprint in English. |
| `pl/fundamentals/` | Polish fundamentals. |
| `pl/key-web-vulnerabilities/` | Polish topic modules. |
| `pl/owasp-top-10-2025/` | Polish OWASP Top 10 2025 mapping sprint. |
| `labs/` folders | Short summaries of legal training labs. |
| `overview.md` files | Concise source-of-truth notes for each topic. |
| `cheat-sheet.md` files | Practical testing, review and remediation checklists. |

## Recommended Path

1. Start with [English notes](eng/README.md) or [Polish notes](pl/README.md).
2. Complete the Web AppSec fundamentals.
3. Work through the Key Web Vulnerabilities topics in order.
4. Use the OWASP Top 10 2025 mapping to connect topics with review-style outputs.
5. Revisit checklists and regression-test notes during future code or AppSec reviews.

## Topic Workflow

Most topics follow the same pattern:

1. Read enough theory to understand the risk.
2. Practise in a legal lab or local test environment.
3. Test manually before relying on automation.
4. Write notes in my own words.
5. Identify the root cause and impact.
6. Write a developer-friendly fix.
7. Add a checklist or takeaway for future reference.

## File Roles

- `README.md` - index, topic scope and navigation.
- `overview.md` - concise source of truth for a topic.
- `cheat-sheet.md` - practical workflow and checklist.
- `labs/*.md` - short summaries of legal training labs.
- `learning-summary.md` or similar - deeper consolidation and personal takeaways.
- `coverage-matrix.md` - OWASP category coverage and next actions.
- `security-findings/*.md` - example finding write-ups for practice.

## Ethics and Scope

All notes and examples in this repository are for learning and professional development.

Practice should be limited to:

- legal labs,
- intentionally vulnerable applications,
- local test environments,
- authorised learning platforms,
- explicitly permitted testing scopes.

Do not test real systems without permission, clear scope and safe harbour.

This repository does not include real target data, credentials, private information, or instructions for testing systems without permission.
