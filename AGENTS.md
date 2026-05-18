# Repository Guidelines

## Project Structure & Module Organization

This repository is a Markdown-based AppSec learning notes project. Content is organized by language and then by learning area:

- `eng/` contains English notes.
- `pl/` contains Polish notes that mirror the English structure where practical.
- `fundamentals/` covers reusable Web AppSec foundations.
- `key-web-vulnerabilities/` contains topic modules such as authentication bypass, IDOR, XSS, SQL injection, and CSRF.
- Topic folders usually contain `README.md`, `overview.md`, `cheat-sheet.md`, optional `summary.md` or `learning-summary.md`, and `labs/` for legal lab summaries.

When adding a new topic, place it under both `eng/key-web-vulnerabilities/` and `pl/key-web-vulnerabilities/` if a translated version is available. Keep indexes updated in the relevant `README.md` files.

### Topic Module Pattern

Keep topic modules consistent with the existing `01-authentication-bypass-username-enumeration` pattern:

- Topic `README.md` should be an index only: start links, topic scope, and file roles.
- Put direct lab links in the topic `README.md`; do not also add a duplicate "Completed Labs" table.
- `overview.md` should be the concise source of truth for concepts, impact, testing approach, and remediation. It may summarise practice areas, but should not repeat the full lab index.
- `cheat-sheet.md` should be a practical testing/review checklist, not another navigation page.
- Use one longer summary file per topic, preferably `learning-summary.md` when matching existing English modules. Do not keep both `summary.md` and `learning-summary.md` for the same topic.
- `labs/` should contain individual `lab-01-...md` files. Do not add `labs/README.md` unless the topic already uses that pattern and there is a clear reason.
- Avoid repeating the same lab list across README, overview, summary, cheat sheet, and lab indexes. Link once from the topic README and keep other files focused on their own purpose.

## Build, Test, and Development Commands

There is no application build pipeline. Use lightweight checks before committing:

- `rg --files` lists tracked content structure quickly.
- `rg "TODO|FIXME"` finds unfinished notes.
- `git diff --check` detects trailing whitespace and common patch issues.
- `git status --short` reviews changed files before commit.

Preview Markdown in your editor or repository UI to verify headings, links, and lists render correctly.

## Coding Style & Naming Conventions

Use Markdown with clear heading hierarchy and short sections. Prefer concise, developer-focused language: root cause, impact, remediation, and review checklist. Use fenced code blocks for request examples, commands, or snippets.

File and folder names should be lowercase kebab-case, for example `04-sql-injection/overview.md` or `lab-01-login-bypass.md`. Keep topic numbering stable once published. Use ASCII punctuation unless the language content requires Polish characters.

## Testing Guidelines

Testing is manual review. Check that internal links resolve, language indexes stay aligned, and lab notes avoid publishing sensitive answers or real target data. For security examples, keep the scope defensive and tied to legal labs, local environments, or authorized learning platforms.

## Commit & Pull Request Guidelines

Recent commits use short conventional-style prefixes such as `docs:`, `feat:`, `refactor:`, and `wip:`. Prefer focused messages like `docs: add csrf cheat sheet` or `refactor: reorganize english notes`.

Pull requests should describe the changed topic, list affected language folders, and call out any new labs or ethical-scope considerations. Include screenshots only when Markdown rendering or diagrams need visual review.

## Security & Configuration Tips

Do not commit credentials, private target data, exploit output from real systems, or instructions for testing without permission. Keep examples educational, scoped, and remediation-oriented.
