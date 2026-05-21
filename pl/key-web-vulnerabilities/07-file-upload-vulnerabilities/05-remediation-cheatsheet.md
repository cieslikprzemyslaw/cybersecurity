# Ściąga z remediacji uploadu plików

## TL;DR

Nie ufaj uploadowanym plikom. Waliduj je po stronie serwera, zapisuj bezpiecznie, zmieniaj nazwy plików i blokuj wykonywanie kodu w katalogach uploadu.

## Kluczowe zabezpieczenia

- walidacja po stronie serwera
- ścisłe allowlisty
- nazwy plików generowane po stronie serwera
- bezpieczny zapis poza wykonywalnym web rootem
- wyłączone wykonywanie kodu w katalogach uploadu
- bezpieczne nagłówki odpowiedzi
- limity rozmiaru
- testy regresji

## Pomysły na testy regresji

```text
shell.php
shell.phtml
shell.phar
shell.php.jpg
shell.jpg.php
shell.php%00.jpg
shell.PHP
SVG ze skryptem
upload HTML
zbyt duże pliki
nazwy plików z path traversal
```

## Główna lekcja

Najmocniejsza poprawka to bezpieczny projekt uploadu: waliduj, zmieniaj nazwy, zapisuj bezpiecznie i uniemożliwiaj wykonanie kodu.
