# In-Band SQL Injection

Ten plik rozwija tylko praktyczną część in-band SQLi. Ogólna klasyfikacja typów jest w [02-types-of-sqli.md](02-types-of-sqli.md).

In-band oznacza, że dowód podatności wraca w tej samej odpowiedzi HTTP, którą testujesz:

```text
request z kontrolowanym inputem
  -> backend wykonuje zapytanie SQL
  -> błąd, zmiana treści albo dane wracają w odpowiedzi
```

## Co sprawdzam w odpowiedzi

Najważniejsze jest to, czy odpowiedź pokazuje, że input zmienił zachowanie zapytania SQL.

Przykładowe sygnały:

- `500 Internal Server Error` po dodaniu apostrofu,
- komunikat `SQL syntax error`,
- nazwa tabeli, kolumny albo silnika bazy w błędzie,
- zmiana liczby wyników,
- dodatkowe dane widoczne w HTML,
- powrót strony do normalnego zachowania po dodaniu komentarza SQL.

## Error-based evidence

Error-based evidence jest najmocniejsze wtedy, gdy odpowiedź ujawnia szczegóły bazy albo pokazuje, że składnia SQL została zepsuta.

W review ważne jest nie tylko "pojawił się błąd", ale:

- czy błąd zależy od kontrolowanego inputu,
- czy wygląda jak błąd bazy danych,
- czy ujawnia szczegóły implementacji,
- czy znika po zmianie payloadu w sposób zgodny z hipotezą SQLi.

## UNION-based evidence

UNION-based evidence jest mocniejsze niż samo potwierdzenie błędu, bo pokazuje realny wpływ: dane z dodatkowego zapytania pojawiają się w normalnej odpowiedzi.

Dobry flow dowodowy:

```text
apostrof psuje odpowiedź
komentarz SQL przywraca odpowiedź
NULL tests ustalają liczbę kolumn
wartości tekstowe pojawiają się w HTML
dane z innej tabeli pojawiają się w HTML
```

Szczegóły techniki UNION są w [04-union-based-sqli.md](04-union-based-sqli.md).

## Dlaczego in-band SQLi jest ważne

In-band SQLi często łatwiej wyjaśnić w raporcie, bo dowód jest widoczny bezpośrednio w odpowiedzi aplikacji.

To pomaga pokazać:

- kontrolowany input,
- zmianę zachowania backendu,
- dane albo błędy widoczne dla użytkownika,
- realny impact.

## Bezpieczna implementacja

- Nie doklejaj inputu użytkownika do stringów SQL.
- Używaj prepared statements / parameterized queries.
- Nie ujawniaj szczegółowych błędów SQL.
- Używaj kont bazy danych z least privilege.
