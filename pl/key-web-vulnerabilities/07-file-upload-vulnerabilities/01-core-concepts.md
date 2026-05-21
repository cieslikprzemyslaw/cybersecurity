# Podatności w uploadzie plików - podstawowe pojęcia

## TL;DR

Podatność w uploadzie plików pojawia się wtedy, gdy aplikacja przyjmuje pliki od użytkownika, ale waliduje, zapisuje, serwuje albo przetwarza je w niebezpieczny sposób.

Najważniejsze pytanie to nie tylko:

> Czy mogę wgrać plik?

Lepsze pytanie AppSec:

> Co aplikacja robi z tym plikiem po wgraniu?

## Dlaczego upload jest ryzykowny

Uploadowany plik to dane kontrolowane przez użytkownika. Użytkownik może kontrolować nazwę pliku, rozszerzenie, `Content-Type` konkretnej części pliku, zawartość, rozmiar, metadane oraz znaki zakodowane lub zaciemnione.

## Główne skutki

- Remote Code Execution
- Stored XSS
- nadpisanie pliku
- path traversal przez nazwę pliku
- ujawnienie informacji

## Model myślenia

Zaakceptowany upload nie oznacza automatycznie wykonania kodu.

Zawsze sprawdzaj:

- finalną nazwę zapisanego pliku
- final URL
- `Content-Type` odpowiedzi
- czy plik jest treścią statyczną
- czy serwer go wykonuje
- czy działa kontrola dostępu

## Główna lekcja

Upload plików staje się niebezpieczny, gdy pliki kontrolowane przez użytkownika są akceptowane, a potem niebezpiecznie zapisywane, serwowane, parsowane albo wykonywane.
