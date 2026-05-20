# File Upload Vulnerabilities

To jest siódmy temat w serii Key Web Vulnerabilities.

## Start Here

1. [Core Concepts](01-core-concepts.md)
2. [Upload Request Anatomy](02-upload-request-anatomy.md)
3. [Validation and Filter Bypass Cheatsheet](03-validation-and-filter-bypass-cheatsheet.md)
4. [Black-Box Testing Methodology](04-black-box-testing-methodology.md)
5. [Remediation Cheatsheet](05-remediation-cheatsheet.md)
6. [TryHackMe Upload Vulnerabilities Summary](labs/tryhackme-upload-vulnerabilities-summary.md)
7. [PortSwigger Lab 01 - RCE via Web Shell Upload](labs/portswigger-01-rce-via-web-shell-upload.md)
8. [PortSwigger Lab 02 - Content-Type Restriction Bypass](labs/portswigger-02-content-type-bypass.md)
9. [PortSwigger Lab 03 - Obfuscated File Extension](labs/portswigger-03-obfuscated-file-extension.md)
10. [PortSwigger Lab 04 - Extension Blacklist Bypass](labs/portswigger-04-extension-blacklist-bypass.md)
11. [PortSwigger Lab 05 - Polyglot Web Shell Upload](labs/portswigger-05-polyglot-web-shell-upload.md)
12. [PortSwigger Lab 06 - Web Shell Upload via Path Traversal](labs/portswigger-06-path-traversal-upload.md)

## Notatki per język i środowisko

- [Apache / Nginx](languages/apache-nginx.md)
- [Cloud Storage](languages/cloud-storage.md)
- [.NET / C#](languages/dotnet.md)
- [Java](languages/java.md)
- [Node.js](languages/nodejs.md)
- [PHP](languages/php.md)

## Pliki testowe do labów

- [README plików testowych](lab-test-files/README.md)
- [Lista nazw plików do testów](lab-test-files/bypass-names/filename-test-list.md)
- [PHP execution marker](lab-test-files/php/php-marker.php)
- [Web shell do labu PHP](lab-test-files/php/php-command-lab-shell.php)

## Topic Focus

- multipart/form-data upload requests
- filename, Content-Disposition and file part Content-Type
- extension, MIME type and magic byte validation
- client-side vs server-side validation
- upload location and execution risk
- RCE, stored XSS, overwrite, path traversal and information disclosure
- developer-focused remediation and regression testing

## Ważna uwaga

Te notatki i pliki testowe są tylko do legalnych labów, środowisk naukowych i autoryzowanego testowania.

## Role plików

- `01-core-concepts.md` wyjaśnia podstawowy mental model.
- `02-upload-request-anatomy.md` pokazuje strukturę requestów multipart upload.
- `03-validation-and-filter-bypass-cheatsheet.md` zbiera labowe checki walidacji.
- `04-black-box-testing-methodology.md` jest praktycznym workflow testowania.
- `05-remediation-cheatsheet.md` zbiera defensywne poprawki i pomysły na regresję.
- `labs/` zawiera krótkie podsumowania legalnych labów.
- `languages/` zawiera notatki o ryzykach i bezpieczniejszych wzorcach per stack.
- `lab-test-files/` zawiera celowo ograniczone pliki wyłącznie do legalnych labów.
