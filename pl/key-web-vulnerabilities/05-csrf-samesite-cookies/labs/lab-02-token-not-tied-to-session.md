# Lab 02 - CSRF Token Not Tied to User Session

Topic: Błędna walidacja CSRF tokena  
Focus: sprawdzenie, czy poprawny token należy do tej samej sesji użytkownika

## Goal

Wykonać atak CSRF w legalnym labie i zmienić adres email ofiary mimo tego, że aplikacja używa CSRF tokenów.

## Input I Controlled

Kontrolowane inputy:

- parametr `email`,
- parametr `csrf`,
- zewnętrzny formularz HTML hostowany na exploit serverze.

## What Happened

Aplikacja dodawała CSRF token do requestu zmiany emaila:

```http
POST /my-account/change-email
Cookie: session=...

email=test@example.com&csrf=TOKEN
```

Na pierwszy rzut oka wyglądało to bezpieczniej niż lab bez ochrony.

Użyłem dwóch kont, żeby sprawdzić, czy token jest powiązany z sesją:

- `wiener:peter`
- `carlos:montoya`

Świeży, nieużyty token z jednego konta mógł zostać zaakceptowany w requestcie z sesją innego konta.

Ważnym sygnałem był response typu "email already in use" zamiast `Invalid CSRF token`. To oznaczało, że walidacja CSRF przeszła i request dotarł do logiki aktualizacji emaila.

Po potwierdzeniu tego użyłem exploit servera z formularzem zawierającym:

```html
<input type="hidden" name="email" value="unique-email@example.com">
<input type="hidden" name="csrf" value="FRESH_UNUSED_TOKEN_FROM_ANOTHER_SESSION">
```

Lab został rozwiązany po użyciu świeżego tokena i unikalnej wartości emaila.

## What I Learned

Samo istnienie CSRF tokena w requestcie nie wystarcza.

Najważniejsze pytanie brzmi:

> Czy ten token należy do tej samej sesji, która wykonuje request?

Jeśli serwer sprawdza tylko, czy token jest poprawny, ale nie sprawdza, czy należy do danej sesji użytkownika, token z jednego konta może zostać użyty przeciwko innemu kontu.

Nauczyłem się też czytać sygnały z odpowiedzi:

- `Invalid CSRF token` oznacza nieudaną walidację tokena,
- `302 /login` zwykle oznacza brakujące lub wygasłe session cookie,
- `email already in use` może oznaczać, że CSRF token został zaakceptowany i request dotarł do logiki biznesowej.

## Impact

W prawdziwej aplikacji atakujący mógłby użyć własnego poprawnego tokena do sfałszowania requestu przeciwko innemu zalogowanemu użytkownikowi, jeśli token byłby akceptowany między sesjami.

To może pozwolić na niechciane zmiany konta mimo pozornej ochrony CSRF.

## Remediation Summary

Zobacz `../overview.md` dla ogólnej sekcji remediation.

Specyficznie dla tego laba:

- wiązać CSRF token z sesją użytkownika,
- odrzucać tokeny wygenerowane dla innej sesji,
- używać nieprzewidywalnych tokenów,
- bezpiecznie rotować i unieważniać tokeny,
- testować cross-session token reuse podczas security review.

## Main Takeaway

CSRF token musi być czymś więcej niż obecnym parametrem. Musi być nieprzewidywalny, walidowany po stronie serwera i powiązany z sesją użytkownika.
