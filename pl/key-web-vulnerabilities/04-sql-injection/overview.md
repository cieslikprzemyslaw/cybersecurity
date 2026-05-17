# SQL Injection - Notatki z nauki

> Status: notatki edukacyjne  
> Zakres: legalne laby i celowo podatne aplikacje  
> Cel: zrozumienie SQL Injection z perspektywy developera i AppSec

## TL;DR

SQL Injection występuje wtedy, gdy dane kontrolowane przez użytkownika są wstawiane do zapytania SQL w niebezpieczny sposób. Dzięki temu użytkownik może zmienić znaczenie zapytania.

Najważniejsze nie jest tylko to, że payload działa. Ważniejsze jest zrozumienie:

- gdzie input trafia do zapytania,
- w jakim kontekście SQL się znajduje,
- jak zmienia logikę query,
- jaki może być wpływ,
- i jak to poprawnie naprawić.

Główna poprawka to nie filtrowanie apostrofów. Główna poprawka to parametryzowane zapytania / prepared statements.

## Co ćwiczyłem

Do tej pory ćwiczyłem SQL Injection w trzech obszarach:

- login bypass,
- manipulacja `WHERE`, żeby zobaczyć ukryte dane,
- użycie `UNION SELECT`, żeby ustalić liczbę kolumn zwracanych przez query.

Celem nie było zapamiętanie payloadów, tylko zrozumienie, jak input użytkownika wpływa na backendową logikę SQL.

## Mental Model

Podczas testowania SQL Injection w labie zadaję sobie pytania:

- Jaki input mogę kontrolować?
- Czy input trafia do SQL query?
- Czy input jest w stringu, liczbie, filtrze albo innym kontekście SQL?
- Czy mogę wyjść z oryginalnego kontekstu?
- Czy mogę zmienić logikę query?
- Czy response pokazuje błąd, redirect, inne dane albo brak widocznej zmiany?
- Czy request blokuje inny mechanizm, np. CSRF albo sesja?

To pomaga nie traktować każdego nieudanego requestu jako nieudanego SQLi.

## Kluczowe obserwacje

### 1. Redirect może oznaczać sukces

W labie z login bypass `302 Found` z redirectem do account page był wskazówką, że logowanie się udało.

### 2. Błąd CSRF/sesji to nie to samo co błąd SQLi

`Invalid CSRF token` nie oznaczał automatycznie, że payload SQLi był zły. Problemem był niedopasowany CSRF token i session cookie.

### 3. Tekst widoczny na stronie nie zawsze jest wykonaną logiką SQL

Jeżeli SQL-podobny tekst pojawia się jako tytuł strony, nie znaczy to jeszcze, że został wykonany jako warunek SQL. Input może nadal być zwykłym tekstem wewnątrz stringa.

### 4. `UNION SELECT` wymaga zgodnej struktury

`UNION SELECT` wymaga tej samej liczby kolumn co oryginalne query oraz kompatybilnych typów danych. `NULL` jest przydatny do testowania liczby kolumn, bo pasuje do wielu typów SQL.

## Błędy, które zauważyłem

- Testowanie złego parametru na początku.
- Dodawanie słów SQL bez wyjścia z oryginalnego kontekstu.
- Traktowanie każdego błędu jako tego samego problemu.
- Patrzenie tylko na widok w przeglądarce zamiast sprawdzać status code, headers, cookies, redirecty i HTML.
- Zapominanie, że `UNION SELECT` wymaga zarówno tej samej liczby kolumn, jak i kompatybilnych typów danych.

## Remediacja developerska

SQL Injection powinno być naprawiane przez oddzielenie kodu SQL od danych użytkownika.

Główna remediacja:

- parameterized queries,
- prepared statements,
- bezpieczne metody ORM tam, gdzie pasują.

Dodatkowe zabezpieczenia:

- walidacja inputu według oczekiwanego typu i formatu,
- minimalne uprawnienia użytkownika bazodanowego,
- generyczne komunikaty błędów,
- logowanie i monitoring podejrzanych zachowań,
- testy regresji,
- code review pod kątem niebezpiecznego budowania query.

Filtrowanie apostrofów nie jest główną poprawką. Bezpieczniejsze podejście to nie budować SQL query przez konkatenację stringów.

## Pytania do code review

- Czy user input jest doklejany do SQL query?
- Czy używane są parametryzowane zapytania?
- Czy input powinien być stringiem, liczbą, enumem albo identyfikatorem?
- Czy user input zmienia logikę query?
- Czy użytkownik może obejść filtry przez zmianę parametrów requestu?
- Czy authorization jest oddzielone od samego filtrowania danych?
- Czy błędy bazy są widoczne dla użytkownika?
- Czy są testy regresji dla nieoczekiwanego lub złośliwego inputu?

## Dalej utrwalam

Chcę jeszcze lepiej utrwalić:

- różnice między error-based, boolean-based, time-based i UNION-based SQLi,
- szybsze rozpoznawanie kontekstu SQL: string, liczba, `WHERE`, `ORDER BY`, `LIKE`,
- proste wyjaśnienie prepared statements,
- rozpoznawanie, czy problem wynika ze składni SQL, CSRF/sesji, walidacji czy typu danych.

## Główna lekcja

SQL Injection to nie tylko payloady. To rozumienie, jak input kontrolowany przez użytkownika zmienia backendową logikę query.

Z perspektywy frontendu wzmacnia to jedną zasadę:

> Frontend nie jest granicą bezpieczeństwa.

Nawet jeśli UI pokazuje tylko dozwolone filtry lub akcje, backend musi bezpiecznie obsłużyć zmodyfikowane requesty.
