# Cheatsheet walidacji i bypassów

## TL;DR

Walidacja uploadu często składa się z kilku słabych checks. Jeden check rzadko wystarcza.

## Client-Side Validation

Client-side validation działa w przeglądarce. Pomaga UX, ale nie jest security boundary.

### Cztery bypassy client-side

1. Wyłącz JavaScript.
2. Zmodyfikuj incoming page response w Burp.
3. Zmodyfikuj upload request w Burp.
4. Wyślij request bezpośrednio do upload endpointu.

## Server-Side Validation

Server-side validation testujemy przez enumerację.

## Extension Validation Test Cases

```text
shell.php
shell.phtml
shell.phar
shell.php5
shell.jpg.php
shell.php.jpg
shell.PHP
```

## Blacklist vs Allowlist

Blacklist blokuje znane złe rozszerzenia. Jest krucha.

Allowlist pozwala tylko na konkretne oczekiwane rozszerzenia, np. `.jpg`, `.jpeg`, `.png`, `.webp`.

## MIME / Content-Type Validation

Słaby serwer może ufać file part Content-Type:

```http
Content-Type: image/png
```

To można zmienić w Burp.

## Magic Bytes

```text
PNG  -> 89 50 4E 47 0D 0A 1A 0A
JPEG -> FF D8 FF DB
```

Magic bytes są lepsze niż samo extension/Content-Type, ale nie są pełną ochroną.

## Obfuscated File Extension Bypass

Legal-lab test cases:

```text
shell.php
shell.php.jpg
shell.jpg.php
shell.php%00.jpg
shell.php%00.png
shell.PHP
shell.pHp
shell.php%2ejpg
shell%2ephp.jpg
```

### Obserwacja z PortSwigger

```text
shell.php         -> rejected
shell.php.jpg     -> accepted, but served as image
shell.jpg.php     -> rejected
shell.php%00.jpg  -> accepted and stored as shell.php
```

## Main takeaway

Upload validation musi być warstwowy. Najważniejsze są safe storage i disabled execution w upload directory.
