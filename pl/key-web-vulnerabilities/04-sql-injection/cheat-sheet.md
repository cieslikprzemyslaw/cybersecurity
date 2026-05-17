# SQL Injection - praktyczna checklista

## 1. Mental model

SQL Injection występuje wtedy, gdy input użytkownika zmienia znaczenie zapytania SQL.

Najważniejsze pytania:

- Jaki input kontroluję?
- Czy input trafia do query SQL?
- Czy jestem w stringu, liczbie, `WHERE`, `ORDER BY`, `LIKE` albo innym kontekście?
- Czy mogę zamknąć obecny kontekst?
- Czy mogę zmienić logikę zapytania?
- Czy response pokazuje błąd, inne dane, redirect albo brak widocznej zmiany?

## 2. Główne typy SQLi

- Error-based - aplikacja ujawnia błędy bazy danych.
- Boolean-based blind - response różni się dla warunku true/false.
- Time-based blind - response opóźnia się dla spełnionego warunku.
- UNION-based - można dołączyć własny wynik przez `UNION SELECT`.
- Authentication bypass - input zmienia logikę sprawdzania credentials.

## 3. Przydatne elementy SQL

Typowe elementy spotykane w labach:

```sql
'
--
AND
OR
UNION SELECT
NULL
ORDER BY
WHERE
```

Komentarz SQL może zakomentować resztę podatnego query:

```sql
'--
```

`NULL` jest często przydatny przy testowaniu liczby kolumn, bo pasuje do wielu typów danych.

## 4. Login bypass pattern

W podatnym formularzu logowania input może zmienić query używane do sprawdzania użytkownika.

Sygnały sukcesu:

- `302 Found`,
- redirect do account page,
- nowe session cookie,
- brak komunikatu błędu,
- response body pokazujące zalogowanego użytkownika.

Uwaga: błędy CSRF albo sesji nie oznaczają automatycznie, że payload SQLi jest zły. Najpierw sprawdź, czy token CSRF i cookie sesyjne są spójne.

## 5. UNION-based workflow

Najpierw ustal liczbę kolumn:

```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

Albo:

```sql
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

Potem znajdź kolumnę kompatybilną z tekstem:

```sql
' UNION SELECT 'test',NULL--
' UNION SELECT NULL,'test'--
```

Dopiero potem próbuj wyciągać dane w legalnym labie.

## 6. Boolean-based blind SQLi

Jeśli aplikacja nie pokazuje błędów, sprawdzaj różnice między warunkiem true i false.

Przykład koncepcyjny:

```sql
' AND 1=1--
' AND 1=2--
```

Szukaj różnic w:

- status code,
- długości odpowiedzi,
- treści,
- liczbie wyników,
- redirectach,
- cookies.

## 7. Time-based blind SQLi

Jeśli response wygląda tak samo, czas odpowiedzi może być sygnałem.

Testuj ostrożnie i tylko w legalnych środowiskach, bo opóźnienia mogą obciążać aplikację.

## 8. Częste błędy

- Testowanie złego parametru.
- Dodawanie słów SQL bez wyjścia z obecnego kontekstu.
- Ignorowanie CSRF/session errors.
- Patrzenie tylko na UI zamiast na surowy response.
- Zapominanie, że `UNION SELECT` wymaga tej samej liczby kolumn.
- Mylenie reflected text z wykonaną logiką SQL.

## 9. Safe testing checklist

- Testuj tylko legalne laby albo uzgodniony scope.
- Nie używaj prawdziwych targetów bez zgody.
- Nie publikuj prawdziwych danych, tokenów ani cookies.
- Zapisuj root cause i remediation, nie tylko payload.
- Potwierdzaj wyniki ręcznie.

## 10. Remediacja

Główna poprawka:

- parameterized queries,
- prepared statements,
- bezpieczne użycie ORM.

Dodatkowe warstwy:

- walidacja typu i formatu inputu,
- least privilege dla użytkownika bazy danych,
- generyczne błędy,
- logging i monitoring,
- testy regresji,
- code review pod kątem konkatenacji query.

Filtrowanie apostrofów nie jest właściwą główną poprawką. Bezpieczne query oddziela kod SQL od danych użytkownika.
