# File Upload Vulnerabilities - Podstawowe koncepty

## TL;DR

File upload vulnerability pojawia się wtedy, gdy aplikacja przyjmuje pliki od użytkownika, ale waliduje, zapisuje, serwuje albo przetwarza je niebezpiecznie.

Najważniejsze pytanie to nie tylko:

> Czy mogę uploadować plik?

Lepsze pytanie AppSec:

> Co aplikacja robi z tym plikiem po uploadzie?

## Dlaczego upload jest ryzykowny

Uploadowany plik to user-controlled input. Użytkownik może kontrolować filename, extension, Content-Type części pliku, content, size, metadata i encoded/obfuscated characters.

## Główne skutki

- Remote Code Execution
- Stored XSS
- File Overwrite
- Path Traversal przez filename
- Information Disclosure

## Mental model

Upload accepted nie oznacza automatycznie code execution.

Zawsze sprawdzaj:

- final stored filename
- final URL
- response Content-Type
- czy plik jest static content
- czy serwer go wykonuje
- czy działa access control

## Main takeaway

File upload staje się niebezpieczny, gdy user-controlled files są akceptowane, a potem niebezpiecznie zapisywane, serwowane, parsowane albo wykonywane.
