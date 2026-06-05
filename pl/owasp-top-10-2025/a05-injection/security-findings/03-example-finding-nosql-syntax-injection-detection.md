# Example Finding: NoSQL Syntax Injection w filtrze produktów

> Przykładowe znalezisko oparte na autoryzowanym labie treningowym. Nazwy hostów, wartości sesji i dane wrażliwe są pominięte.

## Severity

**Medium**

## Summary

Endpoint filtrowania produktów przetwarza input użytkownika wewnątrz MongoDB-style query expression.

Syntax-sensitive input zmienia odpowiedź aplikacji, a kontrolowane warunki true/false dają różne zestawy wyników. To wskazuje, że parametr może zmieniać logikę query, zamiast być traktowany wyłącznie jako dane.

W scenariuszu treningowym warunek always-true spowodował zwrócenie rekordów spoza zamierzonego filtra.

## Affected feature

```http
GET /filter?category=<category>
```

Affected parameter:

```text
category
```

## Evidence

### Baseline

Normalny filtr kategorii zwracał oczekiwane produkty dla wybranej kategorii.

### Syntax-sensitive input

Dodanie pojedynczego apostrofu zmieniło odpowiedź i wskazało, że wartość wpływa na składnię query.

To był użyteczny dowód, ale sam w sobie nie wystarczał. Zachowanie trzeba było porównać z kontrolowanymi warunkami true i false.

### Warunek true

JavaScript-style condition, który był prawdziwy, zwrócił produkty spoza pierwotnego filtra kategorii.

### Warunek false

Analogiczny warunek fałszywy usuwał albo zmieniał oczekiwane wyniki produktów.

Stabilna różnica między odpowiedziami true i false potwierdziła, że input użytkownika może wpływać na backend query expression.

## Attack path

```text
kontrolowany parametr category
-> input trafia do query expression
-> apostrof potwierdza syntax-sensitive context
-> warunek true zmienia zestaw wyników
-> warunek false zmienia zestaw wyników inaczej
-> obejście filtra query
```

## Impact

Atakujący może być w stanie:

- ominąć filtry produktów albo rekordów,
- pobrać ukryte albo niewydane rekordy,
- enumerować dane spoza zamierzonej kategorii,
- użyć różnic odpowiedzi jako punktu startowego do dalszych testów NoSQL Injection.

Udowodniony impact to obejście filtra i niezamierzone ujawnienie danych. Ten finding nie zakłada ekstrakcji credentials, jeśli osobny oracle i dostęp do wrażliwego pola nie zostały potwierdzone.

## Root cause

Aplikacja pozwala, aby input użytkownika stał się częścią wykonywalnej składni NoSQL query.

Prawdopodobne czynniki:

- niebezpieczne budowanie custom query expression,
- konkatenacja stringów podczas budowy filtra,
- brak ścisłej walidacji wartości kategorii,
- zaufanie do linków kategorii wygenerowanych przez frontend,
- szczegółowe albo odróżnialne odpowiedzi dla testów składniowych i boolean.

## Remediation

- Nie doklejaj inputu użytkownika do MongoDB `$where` ani JavaScript-style query expressions.
- Buduj filtry z pól i operatorów wybranych po stronie serwera.
- Waliduj `category` przez allowlistę oczekiwanych identyfikatorów kategorii.
- Nieznane kategorie traktuj jako brak wyników albo kontrolowany błąd walidacji.
- Zwracaj kontrolowane błędy bez szczegółów bazy albo interpretera.
- Dodaj testy regresji dla apostrofu, warunku true i warunku false.

## Verification steps

Po remediacji potwierdź, że:

- apostrof w `category` nie powoduje błędu interpretera,
- warunki true i false są traktowane jako dane albo odrzucane,
- input always-true nie rozszerza zestawu wyników,
- nieznane kategorie nie ujawniają ukrytych rekordów,
- różnice odpowiedzi nie dowodzą już kontroli nad query expression.

## Suggested regression tests

```text
Given category contains a single quote
When the product filter is requested
Then return a controlled response
And do not expose database or JavaScript errors
```

```text
Given category contains an always-true expression
When the product filter is requested
Then do not return products outside the selected category
```

```text
Given category contains paired true and false conditions
When both requests are sent
Then responses do not reveal query-expression control
```
