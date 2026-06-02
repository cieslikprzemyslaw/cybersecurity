# SQLi Filter Bypass and Encoding

## Dlaczego to ważne

Niektóre aplikacje próbują zapobiegać SQL Injection przez usuwanie albo blokowanie określonych znaków lub słów kluczowych.

Przykłady:

- apostrofy i cudzysłowy,
- spacje,
- `OR`,
- `AND`,
- `UNION`,
- `SELECT`.

To jest kruche, bo filtry i parsery SQL mogą interpretować input inaczej.

## Lekcja o encodingu

Jeśli payload pojawia się w wygenerowanym zapytaniu SQL jako literalny zakodowany tekst, na przykład `%20`, `%27` albo `%7C`, może to oznaczać, że nie został zdekodowany na oczekiwanej warstwie.

Ważne rozróżnienie:

```text
Input w UI -> użyj raw payload, jeśli frontend koduje input
Bezpośredni URL / Burp -> użyj zakodowanego payloadu, jeśli trzeba
```

## Scenariusze bez apostrofów

Niektóre filtry usuwają apostrofy albo podwójne cudzysłowy.

Możliwe koncepcje w autoryzowanych labach:

- konteksty numeryczne mogą nie wymagać apostrofów,
- stringi można czasem budować funkcjami takimi jak `CONCAT()` albo `CHAR()`,
- wartości hex mogą reprezentować stringi zależnie od bazy danych.

Celem nie jest zapamiętanie payloadów. Celem jest zrozumienie, czy parser SQL nadal otrzymuje poprawną składnię.

## Scenariusze bez spacji

Jeśli spacje są blokowane albo usuwane, alternatywami mogą być:

- komentarze SQL, takie jak `/**/`,
- tab: `%09`,
- newline: `%0A`,
- form feed: `%0C`,
- carriage return: `%0D`,
- inne warianty whitespace zależnie od parsera.

## Blokowane keywordy

Jeśli słowa kluczowe takie jak `OR` albo `AND` są usuwane, niektóre bazy mogą wspierać alternatywy takie jak `||` dla logiki podobnej do OR.

Dzielenie keywordów komentarzami też może działać w niektórych kontekstach:

```sql
SE/**/LECT
```

To zależy od bazy i zachowania filtra.

## Ważna lekcja AppSec

Blacklisty są słabe, bo atakujący może znaleźć inną poprawną składnię akceptowaną przez parser SQL.

Bezpieczna poprawka to nie silniejsza blacklista.

Bezpieczna poprawka to:

- prepared statements / parameterized queries,
- bezpieczne użycie ORM,
- allowlisty tam, gdzie wartości są ograniczone,
- bezpieczna obsługa błędów,
- testy regresji.

## Bezpieczna nauka

Tych technik należy używać wyłącznie w legalnych labach albo autoryzowanych środowiskach testowych.
