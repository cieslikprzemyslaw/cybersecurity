# Authentication Bypass / Username Enumeration - praktyczna checklista

## Cel

Używaj tej checklisty podczas testowania mechanizmów logowania w legalnych labach, środowiskach treningowych albo systemach, na których masz wyraźną zgodę na testy.

Celem jest sprawdzenie, czy aplikacja zdradza zbyt dużo informacji podczas logowania, resetu hasła, blokady konta albo innych flow związanych z uwierzytelnianiem.

## Co sprawdzać podczas testowania logowania

Zacznij od normalnej próby logowania i przechwyć request w Burp Suite.

Sprawdź:

- endpoint, na przykład `/login`, `/signin` albo `/auth`,
- metodę HTTP,
- format body requestu,
- parametry, które kontrolujesz,
- cookies i zachowanie sesji,
- status code odpowiedzi,
- response body,
- długość odpowiedzi,
- redirecty,
- timing,
- rate limiting,
- zachowanie account lockout.

Typowe kontrolowane inputy:

- username,
- adres email,
- password,
- remember-me value,
- CSRF token,
- headers,
- cookies,
- pola JSON,
- parametry w URL.

## Sygnały username enumeration

Username enumeration może istnieć, jeśli aplikacja zachowuje się inaczej w zależności od tego, czy username istnieje.

Szukaj różnic w:

- widocznych komunikatach błędów,
- interpunkcji, spacjach albo małych zmianach tekstu,
- HTTP status code,
- nagłówku `Location`,
- nagłówku `Set-Cookie`,
- długości odpowiedzi,
- czasie odpowiedzi,
- komunikatach o blokadzie konta,
- zachowaniu rate limit,
- komunikatach resetu hasła,
- krokach MFA,
- flow weryfikacji email.

Przykłady ryzykownego zachowania:

- `Invalid username` dla nieznanych użytkowników,
- `Incorrect password` dla istniejących użytkowników,
- komunikat o blokadzie tylko dla prawdziwych kont,
- wolniejsza odpowiedź tylko dla prawdziwych kont,
- krok MFA tylko po wpisaniu poprawnego username,
- reset hasła, który potwierdza, czy email istnieje.

## Checklista porównywania odpowiedzi

Podczas porównywania poprawnych i błędnych prób sprawdź:

- Czy tekst odpowiedzi jest dokładnie taki sam?
- Czy interpunkcja jest identyczna?
- Czy struktura HTML jest identyczna?
- Czy długość odpowiedzi jest taka sama albo bardzo podobna?
- Czy status code jest taki sam?
- Czy redirecty są takie same?
- Czy cookies są ustawiane tak samo?
- Czy jeden request trwa dłużej od drugiego?
- Czy account lockout pojawia się tylko dla niektórych username’ów?
- Czy rate limiting działa spójnie?

Nie opieraj się tylko na tym, co widać w przeglądarce. Małe różnice w surowym HTML-u mogą zdradzać ważne informacje.

## Praktyczny flow w Burp Repeater

1. Zaloguj się losowym username i losowym password.
2. Wyślij request logowania do Repeatera.
3. Zmieniaj tylko jedną wartość naraz.
4. Porównuj odpowiedzi dla:
   - nieznany username + złe hasło,
   - potencjalnie poprawny username + złe hasło,
   - poprawny kandydat username + testowane hasło.
5. Zapisz każdą różnicę w statusie, długości, body, redirectach, cookies albo timingu.
6. Jeśli odpowiedź wygląda inaczej, potwierdź to kolejnym requestem, zanim uznasz to za prawdziwy sygnał.

Dobry nawyk testowania:

```text
Zmieniaj jedną zmienną naraz.
```

Jeśli username i password zmieniają się jednocześnie, trudniej zrozumieć, co spowodowało różnicę w odpowiedzi.

## Praktyczny flow w Burp Intruder

Użyj Intrudera, gdy musisz przetestować wiele username’ów albo haseł.

Rekomendowany flow:

1. Najpierw testuj username’y:

```text
username=§candidate_username§&password=FixedWrongPassword123
```

2. Szukaj różnic w:

```text
Status | Length | Grep - Extract | Redirect | Set-Cookie
```

3. Po znalezieniu prawdopodobnie poprawnego username testuj hasła:

```text
username=KnownCandidate&password=§candidate_password§
```

4. Szukaj sygnału poprawnego logowania, na przykład:

```text
302 redirect do /my-account
nowe session cookie
brak komunikatu błędu
zmieniona długość odpowiedzi
```

## Przydatne funkcje Burpa

### Grep - Extract

Użyj tego, żeby wyciągnąć komunikat ostrzegawczy z response, na przykład tekst wewnątrz:

```html
<p class=is-warning>...</p>
```

Dzięki temu łatwiej porównać wiele odpowiedzi w Intruderze bez ręcznego czytania całego HTML-a.

### Grep - Match

Użyj tego, żeby wyszukiwać konkretne frazy, na przykład:

```text
Invalid username or password
Too many incorrect login attempts
```

### Comparer

Użyj Comparera do porównania dwóch podejrzanych odpowiedzi. Jest to przydatne, gdy różnica to tylko znak interpunkcyjny, spacja albo mała zmiana w HTML-u.

### Resource Pool

Przy testach account lockout albo timing użyj pojedynczego wątku, jeśli kolejność requestów ma znaczenie.

```text
1 concurrent request
```

Dzięki temu wyniki są łatwiejsze do interpretacji.

## Testowanie account lockout

Account lockout jest mechanizmem bezpieczeństwa, ale może stać się sygnałem username enumeration, jeśli działa inaczej dla istniejących i nieistniejących username’ów.

Sprawdź:

- Czy blokada pojawia się tylko dla istniejących username’ów?
- Czy komunikat potwierdza, że konto istnieje?
- Czy blokada działa po username, IP, sesji albo urządzeniu?
- Czy atakujący może blokować konta innym użytkownikom?
- Czy aplikacja pokazuje czas blokady?
- Czy istnieje logging i alerting?

Schemat testowania:

```text
wyglądający poprawnie username + kilka złych haseł
losowy username + kilka złych haseł
```

Porównaj zachowanie aplikacji.

## Częste błędy

Unikaj tych błędów:

- zmienianie username i password jednocześnie,
- patrzenie tylko na widok w przeglądarce,
- ignorowanie długości odpowiedzi,
- ignorowanie małych różnic w interpunkcji,
- ignorowanie redirectów i cookies,
- zakładanie, że generyczny komunikat zawsze jest bezpieczny,
- zapominanie, że account lockout może zdradzić poprawny username,
- zbyt agresywne testowanie bez zrozumienia rate limitów,
- publikowanie haseł, tokenów, cookies albo bezpośrednich odpowiedzi z labów w notatkach na GitHubie.

## Remediacja dla developerów

Developerzy powinni:

- używać jednego generycznego komunikatu błędu dla logowania,
- utrzymywać taki sam widoczny response dla poprawnych i błędnych username’ów,
- unikać różnych status code dla błędów związanych z username,
- unikać różnych redirectów dla poprawnych i błędnych username’ów,
- ograniczać różnice w długości odpowiedzi,
- ograniczać różnice w czasie odpowiedzi, jeśli to możliwe,
- wdrożyć spójny rate limiting,
- ostrożnie zaprojektować account lockout,
- unikać komunikatów o blokadzie, które potwierdzają istnienie konta,
- logować podejrzane próby logowania,
- alertować przy wzorcach brute-force i credential stuffing,
- rozważyć MFA dla wrażliwych kont,
- testować flow logowania z poprawnymi i błędnymi danymi.

Praktyczny nawyk developerski:

```text
Używaj jednej wspólnej stałej albo template’u dla komunikatów błędów uwierzytelniania.
```

To zmniejsza ryzyko małych różnic, takich jak brakująca kropka, spacja albo niespójne wording.

## Przypomnienie o legalnym i bezpiecznym testowaniu

Testuj tylko tam, gdzie masz zgodę. Brute-force na prawdziwych systemach może być nielegalny, szkodliwy i może zakłócić działanie aplikacji.

Dla prawdziwych aplikacji używaj uzgodnionych kont testowych, zatwierdzonego zakresu, limitów i okien testowych.

## Pytania sprawdzające

- Jakie parametry kontroluję w request logowania?
- Czy aplikacja odpowiada inaczej dla poprawnych i błędnych username’ów?
- Czy komunikat błędu jest naprawdę identyczny?
- Czy status code i redirecty są spójne?
- Czy długość odpowiedzi coś zdradza?
- Czy timing coś zdradza?
- Czy account lockout zdradza, że konto istnieje?
- Czy lockout można nadużyć do blokowania prawdziwych użytkowników?
- Czy rate limiting działa spójnie?
- Czy developer zrozumiałby z mojego raportu, jak to naprawić?

## Główna lekcja

Testowanie authentication to nie tylko szukanie poprawnego hasła. To analiza tego, jak aplikacja zachowuje się, gdy logowanie się nie udaje. Każda powtarzalna różnica może stać się sygnałem dla atakującego.
