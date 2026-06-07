# Example Finding: Server-Side Template Injection w dynamicznym renderowaniu komunikatu

## Summary

Kontrolowany przez uzytkownika parametr `message` jest wlaczany do zrodla szablonu ERB i oceniany po stronie serwera. Atakujacy moze wstrzyknac wyrazenia Ruby template i uzyskac dostep do niebezpiecznych server-side APIs dostepnych dla procesu aplikacji.

## Severity

**High**

Severity zalezy od obiektow, API, sekretow, sciezek filesystemu i uprawnien systemowych dostepnych w template context oraz procesie aplikacji.

## Affected feature

Dynamiczne renderowanie komunikatu out-of-stock.

## Evidence

Unikalny marker wyslany przez `message` zostal odbity w wygenerowanym HTML.

Nastepnie nieszkodliwe wyrazenie arytmetyczne ERB zostalo ocenione i zastapione obliczonym wynikiem. To potwierdzilo, ze wartosc zostala potraktowana jako template syntax, a nie zwykly tekst.

W autoryzowanym labie ten sam injection point dotarl do Ruby `File` API i usunal plik wskazany przez cel labu.

## Impact

Skuteczny atakujacy moze potencjalnie:

- uzyskac dostep do danych z template context,
- ujawnic sekrety aplikacji,
- czytac, modyfikowac albo usuwac pliki dostepne dla procesu,
- wykonywac server-side Ruby code,
- potencjalnie uruchamiac procesy systemu operacyjnego.

Dokladny impact zalezy od runtime exposure i uprawnien procesu.

## Root cause

Aplikacja konstruuje ERB template source z niezaufanego request input. To powoduje, ze dane kontrolowane przez uzytkownika przekraczaja granice dane/kod i staja sie wykonywalna skladnia szablonu.

## Remediation

- Przestan konstruowac ERB source z wartosci requestu.
- Uzyj statycznego szablonu kontrolowanego przez developera.
- Przekazuj display values tylko jako zmienne danych.
- Preferuj staly identyfikator statusu mapowany na komunikat kontrolowany po stronie serwera.
- Minimalizuj obiekty i helpery wystawione do template context.
- Zastosuj least privilege dla procesu aplikacji.
- Zwracaj klientom generyczne template errors.

## Regression tests

- Potwierdz, ze input wygladajacy jak ERB jest zwracany jako tekst albo odrzucany, nigdy oceniany.
- Potwierdz, ze wartosci `message` nie wywoluja filesystem ani process side effects.
- Potwierdz, ze prawidlowe renderowanie out-of-stock nadal dziala.
- Dodaj code-review albo SAST checks dla dynamic template compilation z request data.

## References

- PortSwigger SSTI overview: https://portswigger.net/web-security/server-side-template-injection
- Completed lab: https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic
