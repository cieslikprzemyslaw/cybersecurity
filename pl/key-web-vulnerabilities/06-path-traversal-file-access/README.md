# Path Traversal / File Access Bugs

To szósty temat w serii Key Web Vulnerabilities.

## Start

1. [Core Concepts](01-core-concepts.md)
2. [Where to Test Checklist](02-where-to-test-cheatsheet.md)
3. [Payloads and Bypasses Cheatsheet](03-payloads-and-bypasses-cheatsheet.md)
4. [LFI / RFI Cheatsheet](04-lfi-rfi-cheatsheet.md)
5. [Fixes / Review Cheatsheet](05-remediation-cheatsheet.md)
6. [Lab 01 - File Path Traversal, simple case](labs/portswigger-01-simple-case.md)
7. [Lab 02 - Absolute path bypass](labs/portswigger-02-absolute-path-bypass.md)
8. [Lab 03 - Traversal sequences stripped non-recursively](labs/portswigger-03-non-recursive-stripping.md)
9. [Lab 04 - Superfluous URL decode / double encoding](labs/portswigger-04-double-url-decode.md)
10. [Lab 05 - Validation of start of path](labs/portswigger-05-start-of-path-validation.md)

## Notatki per język

- [Python](languages/python.md)
- [PHP](languages/php.md)
- [Node.js](languages/nodejs.md)
- [Java](languages/java.md)
- [.NET / C#](languages/dotnet.md)

## Zakres tematu

Ten moduł obejmuje:

- file name, file path, relative path, absolute path i canonical path,
- jak `../` może wyjść poza zamierzony katalog bazowy,
- typowe bypassy, takie jak absolute path, nested traversal i double URL encoding,
- różnicę między Path Traversal, LFI i RFI,
- ryzykowne API i bezpieczniejsze wzorce w konkretnych językach,
- praktyczne poprawki i rzeczy do sprawdzenia w review.

## Role plików

- `01-core-concepts.md` wyjaśnia podstawowy mental model.
- `02-where-to-test-cheatsheet.md` jest praktyczną checklistą wyboru miejsc do testowania.
- `03-payloads-and-bypasses-cheatsheet.md` zbiera typowe wzorce payloadów z labów.
- `04-lfi-rfi-cheatsheet.md` wyjaśnia file inclusion i PHP wrappers.
- `05-remediation-cheatsheet.md` zbiera praktyczne poprawki i checki do review.
- `labs/` zawiera krótkie podsumowania legalnych labów PortSwigger.
- `languages/` zawiera ryzykowne API i bezpieczniejsze wzorce per język.
