# SSTI Regression Tests

## Cel

Testy regresji powinny potwierdzac, ze niezaufany input pozostaje danymi i nie moze stac sie skladnia szablonu po poprawce albo przyszlym refactorze.

## Test 1: Wyrazenie pozostaje tekstem

Wyslij nieszkodliwe wyrazenie specyficzne dla engine przez podatny input.

Oczekiwany bezpieczny wynik:

- wyrazenie jest wyswietlone literalnie albo bezpiecznie zakodowane, lub
- aplikacja odrzuca je zgodnie z business rules.

Nieoczekiwany podatny wynik:

- wyrazenie zostaje zastapione obliczona wartoscia.

Model asercji:

```text
input zawiera template expression for 7 * 7
response must not contain 49 as the evaluated replacement
```

Ten test odroznia reflection od template evaluation.

## Test 2: Statyczny szablon nadal dziala

Wyslij normalny dozwolony komunikat albo status.

Oczekiwane:

- poprawny komunikat jest wyswietlany,
- brak template error,
- output encoding nadal dziala.

Security fix nie powinien popsuc legalnego renderowania.

## Test 3: Mapowanie komunikatu po stronie serwera

Gdy aplikacja uzywa identyfikatora statusu:

```text
status=out_of_stock
```

Oczekiwane:

- serwer zwraca staly komunikat out-of-stock.

Dla nieznanej wartosci:

```text
status=<template syntax or unknown key>
```

Oczekiwane:

- safe default albo validation error,
- brak dynamic compilation,
- brak expression evaluation.

## Test 4: Brak side effects

Podaj input z template-like syntax, ktory wywolalby nieszkodliwy test helper w celowo instrumentowanym srodowisku testowym.

Oczekiwane:

- helper nie jest wywolany,
- zaden plik nie jest tworzony, modyfikowany ani usuwany,
- brak outbound request,
- brak child process.

Nie uzywaj destrukcyjnych testow na produkcji.

## Test 5: Error handling

Podaj niepoprawny template-like input.

Oczekiwane:

- brak stack trace,
- brak nazwy pliku szablonu,
- brak engine internals,
- brak ujawnienia sciezki filesystemu,
- chronione logi serwerowe maja wystarczajace szczegoly do debugowania.

## Test 6: Stored input

Gdy wartosc moze byc zapisana w CMS, profilu, bazie albo polu email template:

1. Zapisz tekst wygladajacy jak template syntax.
2. Wyrenderuj kazda funkcje, ktora pozniej go zuzywa.
3. Potwierdz, ze wartosc pozostaje danymi w kazdej sciezce renderowania.

To wykrywa second-order SSTI, gdzie input jest bezpieczny przy zapisie, ale oceniany pozniej.

## Code-level tests

- Aplikacja laduje templates tylko z zatwierdzonych statycznych zasobow.
- Dynamic template compilation z request values jest odrzucane albo flagowane.
- W template context istnieja tylko zatwierdzone variables/helpers.
- Sandbox policy jest weryfikowane, gdy user-authored templates sa celowa funkcja.
- Proces aplikacji ma least-privilege filesystem i network permissions.

## Przykladowe sformulowanie testu

> Given a user-controlled `message` value containing ERB syntax, when the page is rendered, the response must contain the value as text and must not contain the evaluated result or produce any filesystem side effect.
