# NoSQL Injection Learning Summary

## Co zostało ukończone

Ukończyłem teorię NoSQL Injection, ćwiczenia TryHackMe i dwa laby PortSwigger.

Najważniejsze obszary:

- model dokumentów i kolekcji MongoDB,
- query filters,
- Operator Injection,
- Syntax Injection,
- blind extraction przez boolean oracle,
- remediacja i testy regresji.

## Najważniejsze wnioski

- NoSQL Injection nie jest tylko SQLi z inną składnią.
- Struktura requestu ma ogromne znaczenie.
- Wartość może stać się obiektem query.
- Operator Injection często wynika z przyjęcia obiektu tam, gdzie powinien być string.
- Syntax Injection może wystąpić przy custom JavaScript-style query logic.
- Blind extraction wymaga stabilnego markera true/false.
- Background API requests mogą być ważniejsze niż widoczny URL.

## Różnica Operator vs Syntax

Operator Injection:

```text
input -> nested object/operator -> zmienione query
```

Syntax Injection:

```text
input -> wyjście z expression -> dodana składnia -> zmienione query
```

Ta różnica jest ważna przy raportowaniu, bo remediacja i dowody nie zawsze są identyczne.

## Lekcja z labów

W labie wykrywania NoSQL Injection apostrof powodował błąd, a warunek always-true/always-false zmieniał odpowiedź. To potwierdziło Syntax Injection.

W labie ekstrakcji danych podatny request nie był główną stroną konta, tylko endpointem:

```text
/user/lookup?user=wiener
```

To była ważna korekta procesu testowania: trzeba sprawdzać HTTP history i requesty w tle.

## Oracle

W blind extraction nie widzę sekretu od razu. Widzę tylko odpowiedź aplikacji na pytanie:

```text
Czy ten warunek jest prawdziwy?
```

Jeżeli odpowiedź jest stabilna, można ustalić długość sekretu i każdy znak osobno.

## Błędy, które warto zapamiętać

- Nie zakładać, że endpoint logowania jest podatny tylko dlatego, że celem jest hasło.
- Nie zatrzymywać się na widocznym URL, jeśli dane pobiera background request.
- Nie mieszać encodingu bez sprawdzenia, jak parser traktuje request.
- Nie używać automatyzacji przed ręcznym potwierdzeniem oracle.
- W Burp Intruder dobierać attack type do problemu: Cluster Bomb dla kombinacji pozycji i znaków.

## Obecne rozumienie

NoSQL Injection polega na tym, że input zmienia query przez typ, strukturę, operator albo składnię. Bezpieczna implementacja musi utrzymać query structure po stronie serwera, walidować typy i nie traktować inputu klienta jako części wykonywalnego zapytania.
