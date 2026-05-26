# A01 Testy regresji: Broken Access Control

## Cel

Testy regresji powinny udowodnić, że poprawka access control działa i że podatność nie może zostać ponownie wprowadzona.

Dla Broken Access Control nie wystarczy sprawdzić, że request zwraca błąd. Testy powinny również potwierdzić, że underlying application state się nie zmienia.

## Scenariusz docelowy

Ukończony lab pokazał wieloetapowy workflow podnoszenia roli admina, w którym:

- pierwszy krok odrzucał normalnego użytkownika,
- finalny krok potwierdzenia akceptował bezpośredni request normalnego użytkownika,
- normalny użytkownik był podnoszony do administratora.

Poniższe testy regresji mają zapobiec temu wzorcowi.

## Testy pozytywne

### 1. Administrator może wykonać poprawną zmianę roli

```gherkin
Given I am authenticated as an administrator
And a normal user exists
When I complete the role-upgrade workflow
Then the response should indicate success
And the target user's role should be updated
And the administrator should remain authenticated
```

### 2. Autoryzowany workflow nadal działa po poprawce

```gherkin
Given I am authenticated as an administrator
When I use the normal UI flow to upgrade an allowed user
Then the workflow should complete successfully
And the final state should match the expected role change
```

## Testy negatywne

### 3. Normalny użytkownik nie może rozpocząć procesu podnoszenia roli

```gherkin
Given I am authenticated as a normal user
When I send POST /admin-roles with username=<my-username>&action=upgrade
Then the response should be 403 Forbidden
And my role should remain unchanged
```

Uwaga: Niektóre aplikacje mogą zwracać w tym przypadku `401`, ale `403` jest zwykle czytelniejsze, gdy użytkownik jest uwierzytelniony, ale nieautoryzowany.

### 4. Normalny użytkownik nie może wysłać finalnego potwierdzenia bezpośrednio

```gherkin
Given I am authenticated as a normal user
When I send POST /admin-roles with action=upgrade&confirmed=true&username=<my-username>
Then the response should be 403 Forbidden
And my role should remain unchanged
And I should not be able to access /admin
```

### 5. Normalny użytkownik nie może podnieść roli innego użytkownika

```gherkin
Given I am authenticated as a normal user
And another normal user exists
When I send POST /admin-roles with action=upgrade&confirmed=true&username=<other-username>
Then the response should be 403 Forbidden
And the other user's role should remain unchanged
```

### 6. Nieuwierzytelniony użytkownik nie może wykonywać akcji zarządzania rolami

```gherkin
Given I am not authenticated
When I send POST /admin-roles with action=upgrade&confirmed=true&username=<any-username>
Then the response should be 401 Unauthorized or redirect to login
And no role should change
```

## Weryfikacja stanu

Dla każdego testu nieudanej autoryzacji sprawdź:

- response status jest poprawny,
- rola użytkownika się nie zmieniła,
- nie przyznano dostępu admina,
- nie wystąpiła częściowa zmiana stanu,
- użytkownik nie może uzyskać dostępu do `/admin`,
- nie zwrócono wrażliwych danych.

To ważne, bo odpowiedź może wyglądać jak failure, podczas gdy backend nadal zmienia stan.

## Testy zmiany metody i manipulacji requestem

Sprawdź, czy access control jest egzekwowany spójnie po zmianie requestu.

Przykłady:

- zmiana metody HTTP z `POST` na `GET`,
- usunięcie parametrów wyglądających na opcjonalne,
- zmiana kolejności parametrów,
- zmiana `username`,
- zmiana `userId`, jeśli jest używane,
- zmiana `role`,
- replay starego requestu,
- wysłanie tylko finalnego requestu potwierdzającego,
- usunięcie nagłówka `Referer`,
- usunięcie pól używanych tylko client-side.

Oczekiwany wynik powinien pozostać taki sam:

- nieautoryzowani użytkownicy są odrzucani,
- stan się nie zmienia,
- admin access nie jest przyznawany.

## Pomysły na automatyzację

Testy automatyczne można pisać na różnych poziomach.

### API/integration test

Użyj sesji/tokenu normalnego użytkownika i wyślij finalny request potwierdzający bezpośrednio.

Oczekiwany wynik:

- `403 Forbidden`,
- rola bez zmian,
- endpoint admina niedostępny.

### Service/policy test

Przetestuj funkcję autoryzacji bezpośrednio.

Przykładowa logika:

```text
canChangeUserRole(currentUser, targetUser) should return false when currentUser is not an admin.
```

### End-to-end test

Użyj flow przeglądarkowego, aby potwierdzić, że:

- normalni użytkownicy nie widzą UI zarządzania rolami admina,
- bezpośrednie requesty backendowe nadal failują,
- administratorzy mogą ukończyć workflow.

Bezpośredni request backendowy jest kluczowy. Testy tylko UI nie wystarczają.

## Weryfikacja manualna

Kroki weryfikacji manualnej:

1. Zaloguj się jako normalny użytkownik.
2. Przechwyć lub zrekonstruuj finalny request potwierdzający podniesienie roli.
3. Wyślij request bezpośrednio przez Burp Suite albo Postman.
4. Potwierdź, że odpowiedź to `403 Forbidden`.
5. Odśwież sesję albo odpytaj szczegóły konta.
6. Potwierdź, że rola nadal jest normalnym użytkownikiem.
7. Spróbuj wejść na `/admin`.
8. Potwierdź, że dostęp jest odrzucony.

## Pomysły na logowanie i alertowanie

Dla wrażliwych akcji zarządzania rolami rozważ logowanie:

- nieautoryzowanych prób zmiany roli,
- identity użytkownika,
- target username/user ID,
- endpointu,
- timestamp,
- source IP lub metadanych requestu,
- próbowanej akcji.

Nie loguj sekretów, session cookies ani wrażliwych tokenów.

Przyszłe review A09 powinno sprawdzić, czy ten typ nieudanej próby access control jest widoczny dla obrońców.

## Acceptance criteria

Poprawka jest akceptowalna tylko wtedy, gdy:

- każdy endpoint zarządzania rolami egzekwuje server-side authorization,
- finalne kroki potwierdzenia są chronione,
- normalni użytkownicy dostają `403 Forbidden` dla akcji admin-only,
- nieuwierzytelnieni użytkownicy dostają `401 Unauthorized` albo redirect do logowania,
- odrzucone requesty nie zmieniają stanu,
- poprawne workflow administratora nadal działa,
- normalna funkcjonalność użytkownika pozostaje bez zmian,
- testy pokrywają zarówno allowed, jak i denied scenarios.
