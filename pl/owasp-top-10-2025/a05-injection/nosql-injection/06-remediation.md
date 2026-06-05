# NoSQL Injection Remediation

NoSQL Injection należy naprawiać przez kontrolę typów, schematów i struktury query, a nie przez małą blacklistę znaków.

## Zasada główna

```text
Input użytkownika może być wartością.
Input użytkownika nie powinien wybierać struktury query.
```

Backend powinien decydować:

- jakie pola są odpytywane,
- jakie operatory są używane,
- jaki typ ma każda wartość,
- które pola requestu są dozwolone.

## Walidacja schematu

Waliduj request przed budową query.

Przykład oczekiwania:

```text
username: required string
password: required string
rememberMe: optional boolean
```

Odrzucaj:

- obiekty tam, gdzie oczekiwany jest string,
- tablice tam, gdzie oczekiwany jest string,
- nieznane pola,
- zagnieżdżone struktury,
- klucze operatorów takie jak `$ne`, `$regex`, `$where`.

## Bezpieczna budowa query

Niebezpieczny wzorzec:

```text
database.find(request.body)
```

Bezpieczniejszy wzorzec:

```text
validated = validate(request.body)
database.find({ username: validated.username })
```

Różnica jest ważna: w drugim modelu klient podaje wartość, ale nie wybiera struktury filtra.

## Uwierzytelnianie

Nie weryfikuj hasła przez query, szczególnie jako plaintext.

Ryzykowny model:

```text
find user where username == input.username and password == input.password
```

Bezpieczniejszy model:

```text
1. znajdź użytkownika po zwalidowanym username/email
2. pobierz zapisany password hash
3. porównaj podane hasło przez bezpieczną funkcję hash verification
4. zwróć jeden generyczny błąd przy porażce
```

## Operator control

Jeżeli aplikacja naprawdę potrzebuje filtrowania albo sortowania kontrolowanego przez użytkownika:

- porównaj dozwolone operacje z oficjalną referencją MongoDB [Query Predicates](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/),
- użyj allowlisty pól,
- użyj allowlisty operacji,
- mapuj wartości UI na wewnętrzne nazwy pól,
- nie przekazuj surowych filtrów z requestu,
- ogranicz kosztowne regexy i zakresy.

## Unikanie `$where`

`$where` i custom JavaScript query logic są ryzykowne, bo łatwo zmienić wartość w wykonywalne wyrażenie.

Preferuj:

- built-in structured filters,
- konkretne pola,
- konkretne operatory,
- walidowane wartości.

## Obsługa błędów

Produkcja nie powinna ujawniać:

- stack trace,
- błędów MongoDB,
- błędów JavaScript parsera,
- nazw kolekcji,
- szczegółów drivera,
- fragmentów query.

Błędy powinny być kontrolowane i użyteczne dla klienta, ale nie dla atakującego.

## Regression tests

Po poprawce dodaj testy dla:

- zwykłych stringów,
- apostrofów i znaków specjalnych,
- obiektów w miejscu stringa,
- bracket notation,
- operatorów `$ne`, `$regex`, `$where`,
- warunków always-true i always-false,
- różnic odpowiedzi zależnych od sekretu.

## Najważniejszy wniosek

Remediacja jest kompletna dopiero wtedy, gdy:

- input ma egzekwowany typ,
- query structure jest kontrolowana przez backend,
- hasła nie są queryable plaintext,
- błędy nie ujawniają szczegółów interpretera,
- testy regresji obejmują zarówno payloady tekstowe, jak i object-shaped payloads.
