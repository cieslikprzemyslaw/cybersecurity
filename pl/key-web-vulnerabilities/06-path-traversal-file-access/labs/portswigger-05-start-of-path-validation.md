# PortSwigger 05 - Validation of Start of Path

## Co testowałem

Normalny request do obrazka używał pełnej oczekiwanej ścieżki:

```http
GET /image?filename=/var/www/images/46.jpg
```

To sugerowało, że aplikacja oczekuje, aby ścieżka zaczynała się od:

```txt
/var/www/images/
```

## Co znalazłem

Zadziałał payload:

```http
GET /image?filename=/var/www/images/../../../etc/passwd
```

## Dlaczego to ma znaczenie

Input zaczynał się od oczekiwanego base path, ale finalna resolved path wychodziła poza ten katalog.

## Root cause

Aplikacja prawdopodobnie sprawdzała początek stringa, a nie finalną canonical path.

## Wpływ

Atakujący mógł spełnić słaby prefix check i nadal odczytać wrażliwe pliki spoza katalogu obrazków.

## Notatka dla mnie

Prefix check może wyglądać poprawnie na surowym stringu, a resolved path i tak może wskazywać gdzie indziej.
