# A07:2025 - Authentication Failures

## Co oznacza ta kategoria

Authentication odpowiada na pytanie:

> Kim jest ten użytkownik i czy system otrzymał wystarczający dowód jego tożsamości?

Authentication Failure występuje, gdy aplikacja przyjmuje niewystarczający lub nieprawidłowy dowód, pozwala pominąć wymagany krok, ufa kontrolowanym przez użytkownika danym tożsamości, źle zarządza stanem uwierzytelnienia albo udostępnia niebezpieczny mechanizm odzyskiwania konta.

OWASP A07 obejmuje między innymi nieskuteczną ochronę przed automatycznymi atakami na credentials, słabe recovery, brak lub nieskuteczne MFA, session fixation, brak rotacji identyfikatora sesji i nieprawidłowe unieważnianie sesji.

## Najważniejsze rozróżnienia

### Identification

Identification to deklaracja tożsamości, na przykład:

```text
username=carlos
email=user@example.com
userId=123
```

Odpowiada na pytanie: **Jaką tożsamość deklaruje użytkownik?**

Username lub email nie jest dowodem tożsamości.

### Authentication

Authentication weryfikuje dowód, który ma potwierdzić deklarację, na przykład:

- hasło,
- kod MFA,
- password-reset token,
- passkey lub hardware token,
- istniejącą uwierzytelnioną sesję.

Session cookie lub JWT zwykle reprezentuje wcześniejsze uwierzytelnienie. Backend nadal musi go poprawnie zweryfikować.

### Authorization

Authorization decyduje, co uwierzytelniona tożsamość może odczytać lub wykonać.

Użytkownik może być poprawnie uwierzytelniony, ale nadal wykorzystać Broken Access Control, jeżeli aplikacja pozwala mu wykonać akcję poza jego uprawnieniami.

### Accountability

Accountability tworzy audit trail pokazujący, kto wykonał akcję, co się wydarzyło oraz kiedy lub skąd. Logi wspierają wykrywanie i dochodzenie, ale nie zastępują prewencji.

### Session management

Session management utrzymuje stan uwierzytelnienia po loginie. Obejmuje tworzenie, rotację, wygaśnięcie i unieważnienie sesji, sesje równoległe, remember-me oraz logout.

### Password reset a password change

- **Password reset** jest alternatywnym mechanizmem uwierzytelnienia/odzyskiwania, gdy użytkownik nie zna bieżącego hasła.
- **Password change** zwykle odbywa się wewnątrz uwierzytelnionej sesji i często powinien wymagać bieżącego hasła lub recent reauthentication.

### MFA a step-up authentication

- **MFA** wymaga niezależnych czynników w procesie uwierzytelniania.
- **Step-up authentication** wymaga dodatkowego dowodu przed wrażliwą akcją, nawet gdy użytkownik ma już sesję.

## Flow review uwierzytelniania

Dla każdego flow sprawdź:

1. **Identity claim** — które konto jest wskazywane?
2. **Evidence** — co ma udowodnić tożsamość?
3. **Verification** — co musi sprawdzić serwer?
4. **Binding** — czy dowód jest powiązany z poprawnym kontem, akcją i próbą?
5. **Lifetime** — kiedy dowód wygasa?
6. **Single use** — czy można go użyć ponownie?
7. **Session creation** — jaki stan istnieje po każdym kroku?
8. **Session rotation** — czy ID sesji zmienia się po loginie lub zmianie uprawnień?
9. **Invalidation** — co dzieje się po logout, resecie, zmianie hasła lub wyłączeniu konta?
10. **Failure handling** — czy odpowiedzi są generyczne, limitowane, logowane i fail-closed?
11. **Abuse cases** — czy kroki można pominąć, przestawić, powtórzyć lub zmienić?
12. **Regression tests** — jak zachowanie będzie testowane po naprawie?

## Ataki na hasła i credentials

### Username enumeration

Różne zachowanie dla istniejących i nieistniejących kont może ujawnić prawidłowe tożsamości. Sygnały:

- różne komunikaty,
- różne statusy HTTP,
- różne długości odpowiedzi,
- różnice czasowe,
- zachowanie account lock.

Enumeracja tworzy listę celów dla password spraying, credential stuffing, phishingu lub nadużyć recovery.

### Brute force

Brute force wielokrotnie zgaduje credentials dla jednego lub wielu kont.

### Password spraying

Password spraying testuje małą liczbę popularnych haseł na wielu kontach, aby ograniczyć ryzyko blokady jednego konta.

### Credential stuffing

Credential stuffing wykorzystuje pary username/hasło pochodzące z wcześniejszych wycieków.

### Rate limiting i account locking

Jedna kontrola zwykle nie wystarcza:

- IP-only może zostać ominięte przy użyciu rozproszonych źródeł i szkodzić użytkownikom wspólnej sieci.
- blokada tylko per konto może zostać wykorzystana do denial of service.
- permanent lockout może pozwolić atakującemu blokować prawidłowych użytkowników.

Warstwowa ochrona może łączyć limity per konto i per źródło, rosnące opóźnienia, risk signals, MFA, monitoring i powiadomienia.

## Password Reset i Recovery

Reset hasła jest inną drogą do konta i musi być co najmniej tak bezpieczny jak zwykłe logowanie.

Bezpieczny reset token powinien być:

- nieprzewidywalny,
- ograniczony czasowo,
- jednorazowy,
- powiązany z jednym kontem i jednym celem,
- unieważniony po użyciu,
- wysyłany przez zaufany origin i chroniony kanał.

Serwer powinien ustalić konto docelowe na podstawie zaufanego stanu tokenu. Nie powinien ufać osobnemu `username` przesłanemu przez klienta.

Po poprawnym resecie aplikacja powinna rozważyć unieważnienie istniejących sesji, szczególnie gdy reset może oznaczać przejęcie konta.

## Multi-Factor Authentication

MFA jest pełnym przejściem stanu, a nie tylko ekranem wpisania kodu.

Bezpieczny flow rozróżnia:

```text
unauthenticated
    -> password_verified / mfa_pending
    -> fully_authenticated / mfa_completed
```

Przed ukończeniem MFA sesja powinna mieć dostęp wyłącznie do endpointów potrzebnych do zakończenia uwierzytelnienia. Każda chroniona trasa i API musi sprawdzać stan MFA po stronie serwera.

Kody MFA powinny być:

- krótkotrwałe,
- objęte rate limitingiem,
- powiązane z poprawnym użytkownikiem i próbą uwierzytelnienia,
- unieważniane po użyciu, gdy wymaga tego mechanizm.

Backup codes, recovery, trusted devices i wyłączanie MFA nie mogą tworzyć słabszej drogi omijającej MFA.

## Session Management

### Session fixation

Session fixation występuje, gdy atakujący kontroluje lub przewiduje identyfikator sesji przed uwierzytelnieniem, a aplikacja nadal używa go po zalogowaniu ofiary.

### Session rotation

Po poprawnym logowaniu lub zmianie uprawnień serwer powinien wydać nowy, nieprzewidywalny identyfikator sesji. Samo przedłużenie starej sesji anonimowej nie wystarcza.

### Logout

Logout powinien unieważnić sesję po stronie serwera. Samo usunięcie stanu w przeglądarce nie wystarcza.

Praktyczny test: zapisać cookie sesyjne, wylogować się, a następnie ponownie użyć starego cookie na chronionym endpointcie.

### Wygaśnięcie

Sesje powinny posiadać odpowiednie:

- idle timeout,
- absolute timeout,
- lifetime dla remember-me,
- zasady revocation po resecie hasła, zmianie hasła lub wyłączeniu konta.

### Atrybuty cookies

- `HttpOnly` ogranicza bezpośredni dostęp z JavaScript.
- `Secure` ogranicza wysyłanie do HTTPS.
- `SameSite` wpływa na wysyłanie cross-site i wspiera ochronę przed częścią ataków CSRF.
- `Path` i `Domain` nie powinny być szersze niż potrzeba.

To defence in depth. Flagi nie naprawiają braku walidacji tokenu, MFA bypass, session fixation ani braku unieważnienia po stronie serwera.

## Odpowiedzialność i ograniczenia frontendu

Przydatne kontrole frontendowe:

- dobry UX i bezpieczny feedback,
- wczesna walidacja,
- ograniczenie przypadkowych błędów,
- czyszczenie wrażliwego stanu z pamięci,
- unikanie zbędnego przechowywania tokenów,
- bezpieczna obsługa redirectów i błędów.

Jednocześnie:

- React route guard nie jest security boundary.
- Ukryty komponent nie chroni API.
- Disabled button nie egzekwuje rate limitingu.
- Wartości w `localStorage` i `sessionStorage` są kontrolowane przez użytkownika.
- Redirect z chronionej strony nie chroni endpointu.
- Backend musi weryfikować stan uwierzytelnienia dla każdego chronionego requestu.

## Fakty, założenia, dowody i impact

### Fakty

Bezpośrednio zaobserwowane requesty, response'y, cookies, zmiany stanu i udane uwierzytelnienie.

### Założenia

Możliwe wyjaśnienie działania backendu, którego jeszcze nie udowodniono.

### Evidence

Powtarzalny wynik pokazujący, że oczekiwaną kontrolę można ominąć albo że jej brakuje.

### Impact

Dostęp lub akcja możliwa dzięki podatności.

`302` po zmodyfikowanym resecie pokazało przyjęcie requestu. Dopiero zalogowanie się nowym hasłem potwierdziło account takeover.

## Mapowanie STRIDE

Oba ukończone findings mapują się głównie do **Spoofing**, ponieważ pozwalały atakującemu zostać zaakceptowanym jako inna tożsamość:

- brak powiązania reset tokenu z kontem pozwolił zalogować się jako ofiara po zmianie jej hasła,
- brak egzekwowania MFA pozwolił traktować atakującego jako w pełni uwierzytelnionego.

## Security Requirements

Przykładowe wymagania:

- Serwer nie może tworzyć fully authenticated session przed weryfikacją wszystkich wymaganych czynników.
- Password reset token musi być nieprzewidywalny, ograniczony czasowo, jednorazowy i powiązany z jednym kontem.
- Konto resetowane musi wynikać z zaufanego stanu tokenu po stronie serwera.
- Identyfikator sesji musi zostać obrócony po poprawnym uwierzytelnieniu.
- Logout musi unieważnić sesję po stronie serwera.
- Wrażliwe zmiany konta powinny wymagać recent authentication, gdy jest to uzasadnione.
- Odpowiedzi uwierzytelniania nie powinny ujawniać istnienia konta.
- Kontrole muszą być spójnie egzekwowane na wszystkich trasach i API.
