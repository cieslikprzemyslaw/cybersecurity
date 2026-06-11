# TryHackMe - świadomość Python Deserialization

## Źródło

[OWASP Top 10 2025: Insecure Data Handling](https://tryhackme.com/room/owasptopten2025three)

Zakres: Task 4, A08 Software or Data Integrity Failures.

## Cel

Zrozumieć, dlaczego deserializacja niezaufanych danych Python `pickle` jest niebezpieczna.

## Istotny flow

```text
Base64 input kontrolowany przez użytkownika
    -> Base64 decoding
    -> pickle deserialization
    -> rekonstrukcja obiektu
    -> możliwe zachowanie podczas deserializacji
```

## Czego się nauczyłem

- Pickling to serializacja w Pythonie.
- Unpickling rekonstruuje obiekty.
- `__reduce__` może wpływać na sposób rekonstrukcji obiektu.
- Niebezpieczne zachowanie może wystąpić podczas samej deserializacji.
- Base64 zmienia wyłącznie reprezentację.
- Walidacja po natywnej deserializacji może nastąpić zbyt późno.
- Niezaufane dane nie powinny trafiać do `pickle.loads()`.

## Próba i dowód

Wygenerowałem Base64 pickle payload przeznaczony dla laba, ale aplikacja zwróciła:

```text
Deserialization error: UnpicklingError: invalid load key
```

To potwierdza, że przesłane dane nie zostały zaakceptowane jako poprawny pickle w tej ścieżce. Nie potwierdza code execution ani file read.

Zapisuję to jako ćwiczenie awareness, a nie jako samodzielnie udowodniony exploit.

## Główne ryzyko

```text
niezaufane serializowane dane
    -> niebezpieczna natywna deserializacja
    -> rekonstrukcja obiektu kontrolowana przez atakującego
```

## Bezpieczny projekt

- Nie przyjmować pickled objects od niezaufanych klientów.
- Użyć prostego formatu, np. JSON, z restrykcyjnym schematem.
- Parsować wyłącznie oczekiwane primitive values.
- Odrzucać dodatkowe pola i typy.
- Trzymać authority i wrażliwy stan po stronie serwera.
- Weryfikować wymagany stan klienta przed parsowaniem lub rekonstrukcją.
- Stosować least privilege i izolować niebezpieczne przetwarzanie.

## Pomysły na regresję

- Malformed pickle data.
- Nieoczekiwany typ obiektu.
- Dodatkowe pola.
- Unsigned albo modified data.
- Potwierdzenie odrzucenia przed rekonstrukcją obiektu.
- Potwierdzenie braku file, network, command lub state-changing side effects.
