# Path Traversal / File Access Bugs

This is the sixth topic in the Key Web Vulnerabilities series.

## Start Here

1. [Core Concepts](01-core-concepts.md)
2. [Where to Test Checklist](02-where-to-test-cheatsheet.md)
3. [Payloads and Bypasses Cheatsheet](03-payloads-and-bypasses-cheatsheet.md)
4. [LFI / RFI Cheatsheet](04-lfi-rfi-cheatsheet.md)
5. [Fixes / Review Cheatsheet](05-remediation-cheatsheet.md)
6. [Lab 01 - File Path Traversal, Simple Case](labs/portswigger-01-simple-case.md)
7. [Lab 02 - Absolute Path Bypass](labs/portswigger-02-absolute-path-bypass.md)
8. [Lab 03 - Traversal Sequences Stripped Non-Recursively](labs/portswigger-03-non-recursive-stripping.md)
9. [Lab 04 - Superfluous URL Decode / Double Encoding](labs/portswigger-04-double-url-decode.md)
10. [Lab 05 - Validation of Start of Path](labs/portswigger-05-start-of-path-validation.md)

## Language Notes

- [Python](languages/python.md)
- [PHP](languages/php.md)
- [Node.js](languages/nodejs.md)
- [Java](languages/java.md)
- [.NET / C#](languages/dotnet.md)

## Topic Focus

This module covers:

- file names, file paths, relative paths, absolute paths and canonical paths,
- how `../` can escape an intended base directory,
- common bypasses such as absolute paths, nested traversal and double URL encoding,
- the difference between Path Traversal, LFI and RFI,
- language-specific risky APIs and safer patterns,
- practical fixes and review notes.

## File Roles

- `01-core-concepts.md` explains the basic mental model.
- `02-where-to-test-cheatsheet.md` is the practical target-selection checklist.
- `03-payloads-and-bypasses-cheatsheet.md` lists common lab payload patterns.
- `04-lfi-rfi-cheatsheet.md` explains file inclusion concepts and PHP wrapper notes.
- `05-remediation-cheatsheet.md` collects practical fixes and review checks.
- `labs/` contains short legal PortSwigger lab summaries.
- `languages/` contains language-specific risky APIs and safer patterns.
