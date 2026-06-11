# Niebezpieczna deserializacja

## Definicje

Serializacja konwertuje obiekt albo złożoną strukturę danych do formatu nadającego się do przechowywania lub przesyłania.

Deserializacja rekonstruuje obiekt albo strukturę.

```text
obiekt
    -> serializacja
    -> bytes lub tekst
    -> transport/storage
    -> deserializacja
    -> zrekonstruowany obiekt
```

W Pythonie serializacja może być nazywana pickling. W Ruby może być nazywana marshalling.

## Insecure deserialization

Niebezpieczna deserializacja występuje, gdy serializowane dane kontrolowane przez użytkownika są rekonstruowane i traktowane jako zaufane.

Model mentalny:

```text
serializowane dane użytkownika
    -> deserializacja
    -> utworzenie zaufanego obiektu
    -> zachowanie aplikacji
```

Możliwy wpływ:

- manipulacja właściwościami obiektu,
- privilege escalation,
- utworzenie nieoczekiwanego obiektu,
- wywołanie niebezpiecznych metod,
- dostęp do plików,
- denial of service,
- remote code execution.

Należy deklarować wyłącznie wpływ potwierdzony dowodami.

## Dlaczego walidacja po deserializacji jest słaba

Niebezpieczne zachowanie może wystąpić podczas samej rekonstrukcji obiektu. Sprawdzanie obiektu dopiero po deserializacji może nastąpić zbyt późno.

Najsilniejsza kontrola:

> Nie deserializuj niezaufanych natywnych obiektów.

## Rozpoznawanie serializowanych danych

Możliwe wskaźniki:

- czytelna notacja PHP, np. `O:4:"User"...`,
- rozpoznawalna sygnatura danych Java po Base64,
- binarne lub Base64 blobs w cookies,
- Python pickle,
- format sesji specyficzny dla frameworka,
- właściwości obiektu w stanie kontrolowanym przez klienta.

Kodowanie lub brak czytelności nie sprawia, że obiekt staje się zaufany.

## Przebieg analizy

1. Zidentyfikuj serializowaną wartość.
2. Ustal, skąd pochodzi.
3. Odkoduj warstwy reprezentacji bez uznawania ich za zabezpieczenia.
4. Ustal typ obiektu i pola.
5. Znajdź atrybuty istotne z punktu widzenia bezpieczeństwa.
6. Wykonaj jedną kontrolowaną zmianę.
7. Zakoduj poprawnie ponownie.
8. Sprawdź, czy zmieniony stan został zaakceptowany.
9. Oddziel zmianę UI od zachowania chronionego endpointu.
10. Zapisz wyłącznie udowodniony wpływ.

## Bezpieczniejszy projekt

- Unikać natywnej deserializacji niezaufanych danych.
- Używać JSON lub prostego formatu z jawnym schematem.
- Parsować do bezpiecznych primitive types.
- Odrzucać nieoczekiwane pola i typy.
- Nie pozwalać klientowi ustalać autorytatywnych ról, uprawnień, tożsamości ani cen.
- Trzymać wrażliwy stan po stronie serwera.
- Jeśli stan klienta jest konieczny, weryfikować integralność przed parsowaniem lub deserializacją.
- Egzekwować autoryzację endpointów z użyciem zaufanego stanu serwera.
- Stosować expiry i anti-replay, gdy są potrzebne.
- Logować błędy integralności.

## Główna lekcja z laba

Podatność nie brzmiała jedynie:

> Zmieniłem cookie.

Poprawny wniosek:

> Aplikacja deserializowała obiekt PHP kontrolowany przez klienta i ufała zmienionej właściwości `admin` jako stanowi autoryzacji bez wystarczającej ochrony integralności lub weryfikacji po stronie serwera.
