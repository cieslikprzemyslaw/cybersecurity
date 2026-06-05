# NoSQL Injection Cheat Sheet

Używaj tej checklisty podczas nauki, code review albo autoryzowanych testów.

## Szybki model

```text
Czy input jest nadal wartością,
czy stał się częścią query?
```

## Gdzie patrzeć

- login,
- user lookup,
- search,
- filters,
- category parameters,
- profile endpoints,
- JSON APIs,
- URL-encoded forms,
- cookies,
- headers,
- background fetch/XHR requests.

## Pytania o parser

- Czy backend dostaje string, obiekt, tablicę albo nested object?
- Czy body parser zamienia bracket notation w zagnieżdżone obiekty?
- Czy JSON schema odrzuca nieznane pola?
- Czy klucze zaczynające się od `$` są odrzucane?
- Czy frontend ukrywa faktyczny endpoint danych?

## Operator Injection checks

- Czy stringowe pole przyjmuje obiekt?
- Czy `$ne` zmienia wynik?
- Czy `$regex` daje marker true/false?
- Czy login działa, gdy hasło nie jest poprawną wartością?
- Czy query zwraca pierwszy dokument po operator-shaped input?

## Syntax Injection checks

- Czy apostrof zmienia odpowiedź?
- Czy input trafia do `$where` albo custom expression?
- Czy warunek always-true rozszerza wyniki?
- Czy warunek always-false usuwa wynik?
- Czy błędy MongoDB/JavaScript są widoczne?

## Dowody

- baseline normalnego requestu,
- kontrolowany test true,
- kontrolowany test false,
- stabilny marker odpowiedzi,
- rekord spoza zamierzonego filtra,
- różnica długości odpowiedzi,
- błąd składni po apostrofie,
- potwierdzona długość sekretu,
- odzyskane znaki sekretu w legalnym labie.

## Czego unikać w raporcie

- Nie opieraj się tylko na jednym błędzie 500.
- Nie zakładaj bazy danych bez dowodów.
- Nie mieszaj Operator Injection i Syntax Injection, jeśli dowody pokazują tylko jedną formę.
- Nie publikuj prawdziwych sekretów ani danych z realnych systemów.

## Remediacja

- Egzekwuj schemat i typy.
- Odrzucaj obiekty tam, gdzie oczekiwany jest string.
- Nie przekazuj request body bezpośrednio do query.
- Nie pozwalaj klientowi wybierać operatorów query.
- Unikaj `$where` i budowania custom JavaScript query.
- Weryfikuj hasła przez hash verification.
- Normalizuj błędy i odpowiedzi zależne od sekretu.
- Dodaj rate limiting i monitoring.
