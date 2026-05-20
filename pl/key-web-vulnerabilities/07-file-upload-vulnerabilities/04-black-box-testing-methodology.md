# Metodologia black-box testowania uploadu

## TL;DR

Nie zaczynaj od payloadu. Najpierw zrozum, co aplikacja akceptuje, blokuje, zapisuje i serwuje.

## Kroki

1. Rozpoznaj technologię: `Server`, `X-Powered-By`, framework, error messages.
2. Znajdź upload points.
3. Sprawdź client-side validation.
4. Uploaduj bezpieczny poprawny plik.
5. Znajdź finalny URL i miejsce serwowania.
6. Porównaj accepted vs rejected files.
7. Zidentyfikuj filtr: extension, MIME, magic bytes, file size, filename.
8. Sprawdź, czy plik jest static content czy wykonywany.
9. Myśl jak developer: root cause, impact, remediation, regression test.

## Test matrix

```text
test.jpg          -> accepted
shell.php         -> rejected
shell.php.jpg     -> accepted but not executed
shell.jpg.php     -> rejected
shell.php%00.jpg  -> accepted and stored differently
```

## Main takeaway

Black-box file upload testing opiera się na enumeracji i porównywaniu zachowania aplikacji.
