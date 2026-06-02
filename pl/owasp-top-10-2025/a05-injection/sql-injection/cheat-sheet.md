# SQL Injection Cheat Sheet

Ta checklista służy wyłącznie do nauki i autoryzowanych labów.

## Model mentalny

```text
input -> zapytanie SQL -> interpreter bazy danych -> zmienione zachowanie
```

## Wzorce dowodów

```text
Error-based:
Błąd bazy danych pojawia się w odpowiedzi.

Union-based:
Dane z innego SELECT pojawiają się w normalnej odpowiedzi.

Boolean-based blind:
Warunki TRUE i FALSE powodują różne odpowiedzi.

Time-based blind:
Warunek TRUE powoduje mierzalne opóźnienie.

Out-of-band:
Dowód pojawia się przez callback DNS/HTTP/SMB albo zewnętrzny plik/request.
```

## Podstawowy flow testowy

```text
1. Zidentyfikuj kontrolowany input.
2. Wyślij normalny request.
3. Dodaj apostrof i obserwuj zachowanie.
4. Dodaj komentarz SQL i sprawdź, czy zachowanie wraca do normy.
5. Jeśli to lab UNION, ustal liczbę kolumn.
6. Sprawdź kolumny kompatybilne z tekstem.
7. Ostrożnie udowodnij realny impact.
```

## Przypomnienia UNION

```text
UNION SELECT musi zwracać tę samą liczbę kolumn co oryginalne zapytanie.
NULL pomaga testować liczbę kolumn.
Wartości tekstowe, takie jak 'abc','def', pomagają potwierdzić widoczne kolumny tekstowe.
```

## Przypomnienia blind SQLi

```text
Blind SQLi polega na zadawaniu pytań tak/nie.
Znajdź wiarygodny marker.
TRUE -> marker się pojawia
FALSE -> marker znika
```

Przykładowy marker z laba:

```text
Welcome back
```

## Przypomnienia encoding

```text
Raw payload w UI, jeśli frontend koduje input.
Zakodowany payload w URL/Burp przy wysyłaniu bezpośrednim.
Jeśli odpowiedź pokazuje %20 albo %27 literalnie, payload mógł nie zostać zdekodowany zgodnie z oczekiwaniem.
```

## Koncepcje obchodzenia filtrów

- `||` może działać jako alternatywa dla `OR` zależnie od bazy.
- `%0A`, `%09`, `%0C`, `%0D` mogą działać jako alternatywne whitespace.
- Komentarze `/**/` mogą zastępować spacje.
- `CONCAT()`, `CHAR()` albo hex mogą budować stringi bez apostrofów zależnie od bazy.
- Konteksty numeryczne mogą nie wymagać apostrofów.

## Przypomnienia o poprawce

```text
Główna poprawka: prepared statements / parameterized queries.
Kontrole wspierające: allowlisty, walidacja, bezpieczny ORM, least privilege, ogólne błędy, logowanie, testy regresji.
```
