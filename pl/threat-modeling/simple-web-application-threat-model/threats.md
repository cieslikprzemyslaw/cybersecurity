# Rejestr Zagrozen

Wszystkie wpisy sa hipotetyczne i pozostaja otwarte, poniewaz nie testowano realnej implementacji.

## 1. Modified API Request

- **Komponent:** HTTPS API requests
- **Kategoria STRIDE:** Tampering
- **Waznosc:** High
- **Status:** Open

### Opis

Uzytkownik moze zmodyfikowac requesty API wygenerowane przez frontend, zanim dotra do backendu. Dotyczy to wartosci takich jak `userId`, rola, cena albo identyfikatory obiektow.

### Walidacja

Sprawdz, czy backend samodzielnie identyfikuje uwierzytelnionego uzytkownika, weryfikuje uprawnienia, waliduje wszystkie wartosci kontrolowane przez klienta i zwraca tylko dane oraz akcje, do ktorych uzytkownik ma uprawnienia.

Przetestuj zmodyfikowane requesty, aby potwierdzic, ze zmiana `userId`, roli, ceny albo identyfikatorow obiektow nie omija kontroli po stronie serwera.

### Mitygacja

Traktuj kazde pole requestu jako kontrolowane przez atakujacego. Egzekwuj uwierzytelnianie, autoryzacje, walidacje wejscia, sprawdzanie wlasnosci obiektu i reguly biznesowe po stronie backendu. Zwracaj tylko dane wymagane dla uwierzytelnionego uzytkownika i wykonywanej operacji.

## 2. Broken Object-Level Authorization

- **Komponent:** Backend API
- **Kategoria STRIDE:** Elevation of Privilege
- **Waznosc:** High
- **Status:** Open

### Opis

Backend moze pozwalac uzytkownikowi na odczyt albo modyfikacje obiektu innego uzytkownika przez zmiane identyfikatora obiektu w requescie.

### Walidacja

Zweryfikuj, czy backend identyfikuje uwierzytelnionego uzytkownika na podstawie poprawnie zwalidowanego cookie sesyjnego albo access tokenu i nie ufa `userId` przeslanemu przez klienta.

Przetestuj zmienione identyfikatory obiektow w tej samej uwierzytelnionej sesji, aby potwierdzic, ze jeden uzytkownik nie moze uzyskac dostepu do danych innego uzytkownika.

### Mitygacja

Weryfikuj wlasnosc obiektu i uprawnienia przy kazdym requescie. Nie polegaj na ukrytych kontrolkach frontendu ani identyfikatorach uzytkownika przesylanych przez klienta.

## 3. Session or Token Theft

- **Komponent:** Browser
- **Kategoria STRIDE:** Spoofing
- **Waznosc:** High
- **Status:** Open

### Opis

Atakujacy, ktory przejmie cookie sesyjne albo access token, moze podszyc sie pod legalnego uzytkownika.

### Walidacja

Zweryfikuj, czy cookies sesyjne uzywaja atrybutow `Secure`, `HttpOnly` i odpowiedniego `SameSite`. Sprawdz, czy tokeny nie sa przechowywane w `localStorage`, chyba ze istnieje uzasadniony powod projektowy, czy sesje wygasaja poprawnie i czy logout uniewaznia aktywna sesje.

### Mitygacja

Uzywaj `Secure`, `HttpOnly` i odpowiedniego `SameSite` dla cookies. Chron aplikacje przed XSS, rotuj tokeny tam, gdzie ma to sens, ustawiaj wygasanie sesji i uniewazniaj skompromitowane sesje.

## 4. Excessive Data Returned by API

- **Komponent:** API responses
- **Kategoria STRIDE:** Information Disclosure
- **Waznosc:** High
- **Status:** Open

### Opis

API moze zwracac pola albo rekordy, ktorych aktualny uzytkownik nie potrzebuje albo do ktorych nie ma uprawnien.

### Walidacja

Przejrzyj odpowiedzi API i porownaj zwracane pola z tym, czego aplikacja React faktycznie uzywa. Sprawdz, czy API nie ujawnia niepotrzebnych danych osobowych, identyfikatorow wewnetrznych, rol, uprawnien, pol zwiazanych z bezpieczenstwem albo rekordow nalezacych do innych uzytkownikow.

### Mitygacja

Zwracaj tylko wymagane pola, egzekwuj autoryzacje po stronie serwera i uzywaj schematow odpowiedzi albo DTO, aby ograniczyc przypadkowe ujawnianie danych.

## 5. Missing Audit Logging

- **Komponent:** Backend API
- **Kategoria STRIDE:** Repudiation
- **Waznosc:** Medium
- **Status:** Open

### Opis

Wrazliwe akcje moga nie byc zapisywane z wystarczajacymi informacjami, aby ustalic, kto i kiedy je wykonal.

### Walidacja

Sprawdz, czy wrazliwe operacje backendowe tworza uporzadkowane zdarzenia audytowe zawierajace:

- identyfikator uwierzytelnionego uzytkownika
- wykonana akcje
- zasob docelowy
- timestamp w UTC
- request ID albo correlation ID
- wynik: sukces albo porazka
- zrodlowy adres IP, jezeli jest wiarygodny
- uzyteczny kontekst aplikacyjny

Zweryfikuj, czy zwykli uzytkownicy nie moga modyfikowac logow audytowych i czy powiazane zdarzenia da sie sledzic miedzy requestami.

### Mitygacja

Tworz uporzadkowane logi audytowe dla operacji wrazliwych bezpieczenstwa. Domyslnie nie loguj hasel, pelnych tokenow sesyjnych, access tokenow, wrazliwych danych osobowych, danych platniczych ani kompletnych body requestow. Chroń logi przed modyfikacja i zdefiniuj retencje oraz zasady dostepu.

## 6. Sensitive Information Exposed in Browser Console

- **Komponent:** React Frontend
- **Kategoria STRIDE:** Information Disclosure
- **Waznosc:** Medium
- **Status:** Open

### Opis

Aplikacja React moze ujawniac wrazliwe informacje przez `console.log()`, `console.info()`, `console.debug()` albo nieobsluzone komunikaty bledow. Moga to byc odpowiedzi API, tokeny, dane uzytkownika, identyfikatory wewnetrzne, feature flagi albo szczegoly implementacyjne.

### Walidacja

Uzyj SAST i manualnego code review, aby znalezc logowanie wrazliwych danych do konsoli. Uzyj DAST albo manualnego przegladu runtime jako kontroli pomocniczej przez sprawdzenie konsoli przegladarki i bledow produkcyjnych.

### Mitygacja

Usun wrazliwe logowanie debugowe z buildow produkcyjnych. Nigdy nie loguj tokenow, danych uwierzytelniajacych, danych osobowych ani kompletnych odpowiedzi API. Uzywaj kontrolowanych narzedzi logowania i poziomow logow zależnych od srodowiska.

## 7. Injection in Database Query

- **Komponent:** Database queries
- **Kategoria STRIDE:** Tampering
- **Waznosc:** High
- **Status:** Open

### Opis

Dane kontrolowane przez uzytkownika moga trafic do zapytan bazodanowych bez bezpiecznej parametryzacji albo scislej konstrukcji zapytan. Moze to pozwolic atakujacemu zmienic logike zapytania, uzyskac dostep do nieautoryzowanych danych, modyfikowac rekordy, ominac uwierzytelnianie albo wywolac bledy bazy danych.

### Walidacja

Przejrzyj kod dostepu do danych w backendzie i zweryfikuj, ze:

- niezaufane dane wejsciowe nigdy nie sa konkatenowane do stringow zapytan
- przygotowane zapytania albo zapytania parametryzowane sa uzywane konsekwentnie
- surowe zapytania ORM sa zidentyfikowane i przejrzane
- wartosci o ograniczonym zbiorze dozwolonych opcji uzywaja scislych allowlist
- konta bazodanowe stosuja zasade najmniejszych uprawnien
- szczegoly bledow SQL albo bazy danych nie sa ujawniane uzytkownikom
- wejscia NoSQL maja scisle typy i schematy, ktore odrzucaja struktury przypominajace operatory

Uzyj SAST i manualnego code review, aby znalezc niebezpieczna konstrukcje zapytan. Tam, gdzie jest to autoryzowane, przetestuj odpowiednie endpointy kontrolowanymi payloadami injection.

### Mitygacja

Uzywaj zapytan parametryzowanych albo prepared statements. Dla baz NoSQL egzekwuj scisle typy i schematy wejscia, odrzucaj dane przypominajace operatory i unikaj przekazywania obiektow kontrolowanych przez uzytkownika bezposrednio do filtrow zapytan. Stosuj zasade najmniejszych uprawnien dla konta bazy danych.

## 8. Sensitive Data Returned from Database

- **Komponent:** Query results
- **Kategoria STRIDE:** Information Disclosure
- **Waznosc:** Medium
- **Status:** Open

### Opis

Zapytania do bazy danych moga zwracac wiecej rekordow albo pol niz aktualny uzytkownik ma prawo zobaczyc.

### Walidacja

Przejrzyj wybierane pola i zakres zapytan. Sprawdz, czy surowe obiekty bazodanowe nie sa zwracane bezposrednio, zweryfikuj autoryzacje przed pobraniem danych i porownaj zwracane dane z tym, czego API oraz frontend faktycznie potrzebuja.

### Mitygacja

Egzekwuj autoryzacje przed pobraniem danych i po nim, ograniczaj wybierane pola, uzywaj zawężonych zapytan i unikaj zwracania surowych obiektow bazodanowych bezposrednio do klienta.
