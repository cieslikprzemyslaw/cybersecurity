# Authentication Bypass / Username Enumeration

## TL;DR

Username enumeration występuje wtedy, gdy aplikacja zdradza, czy username albo adres email istnieje. Może to nastąpić przez różne komunikaty błędów, długości odpowiedzi, redirecty, timing, cookies albo zachowanie account lockout.

Nie zawsze daje to bezpośredni dostęp do konta, ale pomaga atakującemu skupić dalsze ataki na prawdziwych użytkownikach.

## What I Practised

Przećwiczyłem trzy laby na poziomie beginner-to-medium dotyczące username enumeration:

- oczywiste różnice w odpowiedzi,
- subtelne różnice w odpowiedzi,
- zachowanie account lockout.

Celem nie było tylko ukończenie labów, ale zrozumienie, jak porównywać odpowiedzi logowania, izolować zmienne i rozpoznawać sygnały zdradzające poprawne username’y.

## Key Concept

Authentication bypass i username enumeration to słabości związane z mechanizmem uwierzytelniania. Username enumeration występuje wtedy, gdy flow logowania daje inne informacje zwrotne w zależności od tego, czy username istnieje.

Różnica może być oczywista, na przykład osobne komunikaty dla niepoprawnego username i błędnego hasła. Może też być subtelna, na przykład brakujący znak interpunkcyjny, lekko inna długość odpowiedzi albo account lockout, który pojawia się tylko dla istniejących kont.

## What the User Controls

W tych ćwiczeniach główne kontrolowane inputy to:

- `username`,
- `password`,
- parametry w request body,
- kolejność i powtarzanie requestów,
- timing i liczba prób logowania.

W prawdziwej aplikacji znaczenie mogą mieć też inne kontrolowane elementy, takie jak headers, cookies, pola JSON, parametry URL i wartości formularza resetu hasła.

## Where the Input Goes

Username i password są przetwarzane przez backendową logikę uwierzytelniania.

Backend zwykle sprawdza:

- czy username istnieje,
- czy password pasuje,
- czy konto jest zablokowane,
- czy obowiązuje rate limit,
- czy należy utworzyć sesję.

Jeśli aplikacja zwraca różne odpowiedzi dla różnych gałęzi tej logiki, te różnice mogą wyciekać informacje.

## Impact

Username enumeration może pomóc atakującemu zidentyfikować prawdziwe konta i skupić na nich dalsze ataki.

Realistyczny impact obejmuje:

- targeted password guessing,
- password spraying,
- credential stuffing,
- phishing skierowany do znanych użytkowników,
- nadużycie account lockout,
- zmniejszenie wysiłku potrzebnego do prób przejęcia konta.

Ten problem zwykle nie ujawnia haseł bezpośrednio, ale ułatwia i ukierunkowuje kolejne ataki.

## Developer Remediation

Developerzy powinni zapewnić spójne zachowanie przy nieudanych próbach logowania.

Rekomendowane zabezpieczenia:

- używać jednego generycznego komunikatu, na przykład `Invalid username or password`,
- unikać osobnych komunikatów dla błędnego username i błędnego password,
- utrzymywać spójne status code dla nieudanych logowań,
- unikać różnych redirectów dla poprawnych i błędnych username’ów,
- unikać oczywistych różnic w długości odpowiedzi,
- ograniczać różnice w czasie odpowiedzi, jeśli to możliwe,
- wdrożyć spójny rate limiting,
- zaprojektować account lockout tak, aby nie potwierdzał istnienia konta,
- nie pokazywać komunikatów o blokadzie tylko dla poprawnych username’ów,
- dodać logging i alerting dla podejrzanych prób logowania,
- rozważyć MFA dla wrażliwych kont,
- testować logowanie, reset hasła i MFA z poprawnymi oraz błędnymi użytkownikami.

Dobry nawyk implementacyjny to użycie jednej wspólnej stałej albo template’u dla komunikatów o nieudanym logowaniu. Pomaga to uniknąć przypadkowych różnic w treści, spacjach albo interpunkcji.

## Review Checklist

- Czy formularz logowania zdradza, czy username istnieje?
- Czy odpowiedzi dla błędnego username i błędnego password są identyczne?
- Czy w HTML-u są małe różnice tekstowe?
- Czy długości odpowiedzi są różne?
- Czy redirecty są różne?
- Czy cookies są ustawiane inaczej?
- Czy timing różni się między poprawnym i błędnym username?
- Czy account lockout uruchamia się tylko dla istniejących użytkowników?
- Czy mechanizm blokady można nadużyć do blokowania prawdziwych użytkowników?
- Czy rate limiting działa spójnie?
- Czy nieudane próby logowania są logowane i monitorowane?

## Main Takeaway

Flow logowania nie powinien zdradzać, która część próby uwierzytelnienia była poprawna. Nawet małe różnice w komunikacie, długości odpowiedzi, timingu albo account lockout mogą pomóc atakującemu zidentyfikować poprawne konta.
