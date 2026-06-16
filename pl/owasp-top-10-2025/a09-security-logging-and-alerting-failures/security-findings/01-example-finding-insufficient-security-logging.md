# Niewystarczające logowanie i alertowanie zmian uprawnień

## Severity

Medium

Ostateczna severity zależy od modelu uprawnień aplikacji, wrażliwości danych, istniejących kontroli administracyjnych i innych źródeł monitoringu. W środowisku wysokiego ryzyka albo regulowanym może być wyższa.

## Kategoria

- OWASP Top 10 2025: A09 Security Logging and Alerting Failures
- Powiązane obszary: accountability, privileged access, incident detection, forensic readiness
- Powiązane weakness patterns: omission of security-relevant information i insufficient logging

## Podsumowanie

Aplikacja pozwala administratorowi zmienić rolę innego użytkownika, ale przeanalizowany przepływ nie tworzy kompletnego, chronionego audit eventu zawierającego aktora, cel, poprzednią rolę, nową rolę, timestamp i result.

Nie potwierdzono również reguły detekcji ani alertu dla nieoczekiwanych lub high-risk privilege changes.

W rezultacie nieautoryzowana, wykonana przez przejęte konto albo przypadkowa zmiana roli może nie zostać wykryta na czas, a późniejsze dochodzenie może nie ustalić, kto wykonał zmianę i jaki był wcześniejszy stan.

## Dotknięty przepływ

```text
administrator request
    -> role-change endpoint
    -> database update
    -> successful response
    -> missing or incomplete audit event
    -> no verified detection or alert
```

## Evidence

W sanitised review scenario:

- operacja role change zwróciła sukces,
- pierwotnie proponowany log zawierał niebezpieczne pola, takie jak session cookie, access token i pełne request body,
- actor i target internal IDs zostały początkowo pominięte w minimalnym projekcie audytu,
- nie było dowodu na correlation rule, alert ownera ani response playbook dla podejrzanych privilege changes.

To przykładowe znalezisko oparte na ukończonym ćwiczeniu projektowym, a nie twierdzenie dotyczące produkcyjnego systemu.

## Wpływ

Złośliwy albo przejęty administrator może zmieniać uprawnienia z mniejszym prawdopodobieństwem szybkiej detekcji.

Możliwe konsekwencje:

- nieautoryzowany dostęp administracyjny,
- dostęp do wrażliwych danych lub funkcji,
- utrata accountability,
- opóźniony incident response,
- niekompletne forensic evidence,
- trudności regulacyjne lub audytowe.

Logging failure samo w sobie nie przyznaje uprawnienia. Zmniejsza visibility i zdolność wykrycia oraz zbadania nadużycia funkcji zmiany uprawnień.

## Root cause

Wymagania dotyczące security events nie zostały zdefiniowane jako część privileged workflow. Implementacja koncentrowała się na zmianie w bazie danych, ale nie określiła:

- authoritative event schema,
- sensitive-data exclusions,
- protected audit storage,
- detection logic,
- alert ownership,
- response requirements.

## Wymaganie bezpieczeństwa

> Każda privileged role albo permission change musi utworzyć chroniony audit event zawierający internal ID aktora, internal ID target usera, poprzednią wartość, nową wartość, timestamp, result i correlation identifier.

Dodatkowe wymagania:

> Authentication secrets, session cookies, access tokens, passwords i pełne request bodies nie mogą być zapisywane w audit evencie.

> Unexpected albo high-risk privilege changes muszą być objęte udokumentowaną detection rule, alert ownerem i response playbookiem.

## Rekomendowana naprawa

### Event generation

- Emitować structured event z backend service, która egzekwuje i wykonuje role change.
- Używać application-controlled event name, np. `privilege_permissions_changed`.
- Zapisywać actor, target, previous role, new role, timestamp, result i request ID.
- Zapisywać failed i denied role-change attempts tam, gdzie wspierają detekcję.

### Sensitive-data controls

- Budować event z allowlist zatwierdzonych pól.
- Nie logować całego request body.
- Wykluczyć passwords, tokens, cookies i authorization headers.
- Używać internal IDs zamiast zbędnych danych osobowych.

### Protection i detection

- Forwardować event do chronionego central collection.
- Ograniczyć read, write, modification i deletion permissions.
- Zdefiniować detekcję dla unexpected administrator assignment, powtarzających się denied changes, emergency accounts i zmian poza normalnym procesem.
- Przypisać alert ownership i udokumentować response.

## Testy regresji

1. Wykonać udaną zmianę roli i potwierdzić jeden kompletny audit event.
2. Potwierdzić actor ID, target ID, previous role, new role, timestamp, result i request ID.
3. Potwierdzić brak password, cookie, access token i full request body.
4. Spróbować zmiany bez uprawnienia i potwierdzić denial oraz odpowiedni security event.
5. Wysłać newline/control characters w logowalnym polu i potwierdzić jeden poprawny structured event.
6. Spróbować zmienić lub usunąć audit record i potwierdzić denial albo detection.
7. Uruchomić zdefiniowany suspicious-role-change condition i potwierdzić dostarczenie alertu do ownera.

## Developer takeaway

Udana aktualizacja bazy danych nie jest wystarczającym audit trail. Privileged workflows wymagają świadomie zaprojektowanych dowodów, ochrony, detekcji, ownership i response.
