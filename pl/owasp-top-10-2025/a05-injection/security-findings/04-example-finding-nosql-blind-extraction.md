# Example Finding: NoSQL Syntax Injection Allows Blind Extraction of Administrator Credentials

> Przykładowe znalezisko oparte na autoryzowanym labie treningowym. Nazwy hostów, wartości sesji i credentials są pominięte.

## Severity

**High**

## Summary

Endpoint user lookup buduje zapytanie MongoDB z użyciem inputu użytkownika wewnątrz JavaScript-style expression.

Atakujący może wyjść z zamierzonej wartości username, dodać warunki boolean i obserwować różne odpowiedzi zależnie od tego, czy wstrzyknięty warunek jest prawdziwy.

Takie zachowanie tworzy boolean oracle, który pozwala ustalać wartości wrażliwych pól znak po znaku. W scenariuszu treningowym hasło administratora zostało odzyskane i użyte do dostępu do konta administratora.

## Affected feature

```http
GET /user/lookup?user=<username>
```

Affected parameter:

```text
user
```

## Evidence

### Baseline

Normalny lookup zwrócił żądanego użytkownika:

```json
{
  "username": "wiener",
  "email": "wiener@normal-user.net",
  "role": "user"
}
```

### Syntax-sensitive input

Dodanie apostrofu zmieniło odpowiedź na:

```json
{
  "message": "There was an error getting user details"
}
```

To wskazywało, że parametr wpływa na składnię query.

### Warunek always-true

JavaScript-style always-true condition spowodował, że endpoint zwrócił rekord administratora zamiast żądanego zwykłego użytkownika.

### Boolean oracle

Warunek sprawdzający długość hasła zwracał rekord administratora tylko wtedy, gdy testowana długość była poprawna.

Warunki dla pozycji znaków dawały tę samą różnicę true/false, co pozwalało zrekonstruować hasło.

## Attack path

```text
kontrolowany parametr user
-> wyjście z wyrażenia query
-> wstrzyknięcie warunku boolean
-> identyfikacja markera true/false
-> ustalenie długości hasła
-> test każdej pozycji znaku
-> odzyskanie hasła administratora
-> przejęcie konta administratora
```

## Impact

Nieuwierzytelniony albo niskouprawniony atakujący z dostępem do endpointu może być w stanie:

- pobierać rekordy spoza zamierzonego lookupu,
- enumerować konta uprzywilejowane,
- wnioskować wartości wrażliwych pól,
- ekstrahować credentials albo tokeny,
- przejąć konta administratorów,
- uzyskać dostęp do funkcji administrator-only.

Impact zależy od pól dostępnych dla query, wymagań uwierzytelniania endpointu i tego, czy wartości wrażliwe są przechowywane w formie możliwej do odpytywania.

## Root cause

Aplikacja pozwala, aby input użytkownika stał się częścią wykonywalnej składni MongoDB/JavaScript query.

Prawdopodobne czynniki:

- niebezpieczne użycie `$where` albo custom JavaScript query logic,
- konkatenacja stringów podczas budowy query,
- brak ścisłej walidacji inputu,
- wrażliwe dane haseł możliwe do odpytywania,
- różne odpowiedzi tworzące stabilny boolean oracle,
- niewystarczający rate limiting.

## Remediation

### Query construction

- Usuń budowanie custom JavaScript query tam, gdzie to możliwe.
- Nie doklejaj inputu użytkownika do `$where` ani żadnego wykonywalnego wyrażenia.
- Używaj wbudowanych structured filters z polami i operatorami wybranymi po stronie serwera.
- Egzekwuj, że `user` jest prymitywnym stringiem.

### Authentication and password storage

- Przechowuj hasła z użyciem odpowiedniego algorytmu haszowania haseł.
- Pobieraj użytkowników tylko po dokładnym, zwalidowanym username albo email.
- Weryfikuj podane hasło względem zapisanego hasha poza query do bazy.
- Nigdy nie odpytuj plaintext password values.

### Response and abuse controls

- Zwracaj kontrolowane, niespecyficzne błędy.
- Unikaj różnic odpowiedzi, które ujawniają wynik warunków zależnych od sekretów.
- Ograniczaj powtarzalne próby lookupu przez rate limiting.
- Monitoruj apostrofy, operatory JavaScript, powtarzane testy długości i próbkowanie pozycji znaków.

## Verification steps

Po remediacji potwierdź, że:

- apostrof nie powoduje błędu interpretera ani aplikacji,
- JavaScript expressions są traktowane jako dane albo odrzucane,
- warunek always-true nie zwraca innego użytkownika,
- warunki długości i znaków hasła nie wpływają na odpowiedzi,
- zagnieżdżone albo niestringowe wartości są odrzucane,
- wrażliwe dane hasła nie są queryable,
- rate limiting i monitoring wykrywają automatyczne próbkowanie.

## Suggested regression tests

```text
Given user contains a quote
When lookup is requested
Then return a controlled validation/not-found response
And do not expose database errors
```

```text
Given user contains JavaScript boolean syntax
When lookup is requested
Then do not broaden the result set
And do not return another user's record
```

```text
Given repeated secret-dependent conditions
When requests are sent
Then responses do not reveal condition truth
And abuse controls are triggered
```
