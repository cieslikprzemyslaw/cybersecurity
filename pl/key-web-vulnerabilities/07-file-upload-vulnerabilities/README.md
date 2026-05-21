# Podatności w uploadzie plików

To jest siódmy temat w serii kluczowych podatności webowych.

## Zacznij tutaj

1. [Podstawowe pojęcia](01-core-concepts.md)
2. [Anatomia requestu uploadu](02-upload-request-anatomy.md)
3. [Ściąga z walidacji i bypassów filtrów](03-validation-and-filter-bypass-cheatsheet.md)
4. [Metodologia testowania black-box](04-black-box-testing-methodology.md)
5. [Ściąga z remediacji](05-remediation-cheatsheet.md)
6. [TryHackMe Upload Vulnerabilities - podsumowanie](labs/tryhackme-upload-vulnerabilities-summary.md)
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

## Zakres tematu

- requesty uploadu `multipart/form-data`
- nazwa pliku, `Content-Disposition` i `Content-Type` konkretnej części pliku
- walidacja rozszerzeń, MIME type i magic bytes
- walidacja po stronie klienta i po stronie serwera
- miejsce zapisu uploadu i ryzyko wykonania kodu
- RCE, stored XSS, nadpisanie pliku, path traversal i ujawnienie informacji
- remediacja oraz testy regresji z perspektywy developera

## Ważna uwaga

Te notatki i pliki testowe są tylko do legalnych labów, środowisk naukowych i autoryzowanego testowania.

## Role plików

- `01-core-concepts.md` wyjaśnia podstawowy model myślenia.
- `02-upload-request-anatomy.md` pokazuje strukturę requestów `multipart/form-data`.
- `03-validation-and-filter-bypass-cheatsheet.md` zbiera labowe testy walidacji.
- `04-black-box-testing-methodology.md` jest praktycznym procesem testowania.
- `05-remediation-cheatsheet.md` zbiera defensywne poprawki i pomysły na regresję.
- `labs/` zawiera krótkie podsumowania legalnych labów.
- `languages/` zawiera notatki o ryzykach i bezpieczniejszych wzorcach dla konkretnych stacków.
- `lab-test-files/` zawiera celowo ograniczone pliki wyłącznie do legalnych labów.
