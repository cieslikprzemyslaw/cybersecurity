# A07 Learning Notes

## Punkt początkowy

Rozumiałem już podstawowe flow loginu, cookies, tokeny, username enumeration oraz ogólną różnicę między authentication i authorization.

Celem nie było zostanie identity architect. Chodziło o praktyczną podstawę do review typowych flow jako Frontend Developer przechodzący do AppSec.

## Co od początku rozumiałem dobrze

- Identification wybiera lub deklaruje tożsamość.
- Authentication dostarcza i weryfikuje dowód.
- Authorization określa dozwolone działania.
- Accountability rejestruje, kto co zrobił i kiedy.
- Client-side controls nie wystarczają do ochrony authentication.

## Korekty, które poprawiły moje rozumienie

### Username jest deklaracją, a nie dowodem

Początkowo opisywałem identity głównie jako ID lub username. Ważna korekta:

> Twierdzę, że jestem tym kontem.

Dopiero hasło, MFA, recovery token lub poprawna sesja stanowi dowód, który musi sprawdzić backend.

### Cookie nie staje się automatycznie sesją innego użytkownika

W labie resetu hasła początkowo zastanawiałem się, czy browser `session` cookie może stać się sesją Carlosa, ponieważ request zawierał `username=carlos`.

Kontrolowany request do `/my-account` nie potwierdził tej hipotezy. Cookie reprezentowało bieżącą sesję przeglądarki, a nie dowód uwierzytelnienia Carlosa.

To pomogło oddzielić:

- hipotezę dotyczącą cookie,
- od prawdziwego recovery evidence znajdującego się w reset linku.

### Po loginie potrzebna jest rotacja, a nie tylko czasowa ważność

Początkowo odpowiedziałem, że sesja powinna zostać przedłużona lub otrzymać czasową ważność.

Ważna korekta:

- backend powinien wydać lub obrócić nowy nieprzewidywalny session ID,
- powiązać sesję z uwierzytelnionym użytkownikiem,
- ustawić odpowiedni lifetime.

Chroni to przed kontynuacją sesji anonimowej lub kontrolowanej przez atakującego.

### Test logoutu to replay starej sesji

Najpierw opisałem to jako próbę ponownego logowania tym samym tokenem. Precyzyjny test:

1. zapisz authenticated session,
2. wykonaj logout,
3. użyj zapisanej sesji na chronionym endpointcie,
4. potwierdź odrzucenie przez backend.

### MFA jest przejściem stanu, a nie stroną

Lab 2FA pokazał najważniejszą rzecz:

- aplikacja pokazała stronę MFA,
- ale backend pozwolił sesji po samym haśle wejść na `/my-account`.

Problemem nie była sama możliwość zmiany URL. Serwer nie egzekwował ukończenia MFA.

Lepszy model:

```text
unauthenticated
    -> password_verified / mfa_pending
    -> fully_authenticated / mfa_completed
```

### Password reset jest alternatywnym authentication

Początkowo nie potrafiłem wyjaśnić, dlaczego MFA nie naprawia podatnego resetu.

Korekta: reset hasła to osobna droga potwierdzania dostępu do konta. Jeżeli recovery evidence jest słabe lub źle powiązane, MFA w zwykłym loginie nie naprawia tej ścieżki.

## Tok rozumowania w labie Password Reset

### Zamierzony flow

1. Użytkownik prosi o reset.
2. Serwer wystawia token dla tego konta.
3. Użytkownik otrzymuje token kanałem recovery.
4. Serwer weryfikuje token.
5. Serwer resetuje wyłącznie konto powiązane z tokenem.

### Co kontrolowałem

```text
temp-forgot-password-token=<valid-token-for-wiener>
username=carlos
new-password-1=<new-password>
new-password-2=<new-password>
```

### Rozwój dowodów

- `302` pokazało przyjęcie requestu.
- Nie potwierdzało jeszcze account takeover.
- Udany login jako Carlos nowym hasłem potwierdził impact.

### Finalne rozumienie

Root cause nie polegał na tym, że hidden field można edytować. Wszystkie dane klienta można edytować.

Backend nie sprawdził, czy recovery evidence autoryzuje reset wybranego konta.

## Tok rozumowania w labie 2FA

### Zamierzony flow

1. Zweryfikuj username i hasło.
2. Utwórz ograniczony stan MFA-pending.
3. Zweryfikuj drugi czynnik.
4. Dopiero wtedy udostępnij chronione zasoby.

### Zaobserwowane zachowanie

Po poprawnych credentials, ale bez kodu MFA, sesja mogła otworzyć `/my-account` jako Carlos.

### Finalne rozumienie

Frontend wyświetlał drugi krok, ale chroniony zasób nie sprawdzał ukończenia MFA.

## Fakty a założenia

W obu labach ważne było, aby nie deklarować impactu zbyt wcześnie.

### Password reset

- Fakt: serwer odpowiedział `302` na zmieniony request.
- Założenie: hasło Carlosa mogło zostać zmienione.
- Evidence: login jako Carlos nowym hasłem zadziałał.
- Impact: potwierdzony account takeover.

### MFA

- Fakt: po haśle istniała sesja.
- Observation: pokazano ekran MFA.
- Evidence: `/my-account` zwróciło konto Carlosa bez kodu.
- Impact: potwierdzony MFA bypass dla atakującego z poprawnymi credentials.

## Findings, które potrafię wyjaśnić

### Password reset

> Atakujący znający prawidłowy username może użyć własnego reset tokenu do zmiany hasła innego konta, ponieważ backend nie sprawdza powiązania tokenu z wybranym użytkownikiem. Może to prowadzić do account takeover, również kont uprzywilejowanych.

### MFA

> Atakujący posiadający prawidłowy username i hasło może ominąć MFA, ponieważ aplikacja daje dostęp do chronionych zasobów przed potwierdzeniem drugiego czynnika przez backend.

## Obecne ograniczenia

Rozumiem podejście review i podstawowe kontrole, ale nie ukończyłem osobnych labów A07 dla każdego wzorca session management i credential attacks.

Dalsza praktyka może pogłębić:

- session fixation,
- revocation sesji na wielu urządzeniach,
- bezpieczeństwo remember-me,
- brute force kodów MFA,
- złożone bypassy rate limitingu,
- walidację specyficzną dla SSO/OIDC.

To kolejny poziom, a nie blocker dla obecnej podstawy FE -> AppSec.

## Interview-style Takeaways

- Identification deklaruje tożsamość; authentication weryfikuje dowód; authorization sprawdza uprawnienia.
- Recovery token jest dowodem uwierzytelnienia i musi być powiązany z jednym kontem.
- MFA musi być egzekwowane stanem backendu na każdym chronionym zasobie.
- React route guard wspiera UX, ale nie jest security boundary.
- Session ID powinien rotować po loginie, a logout powinien unieważniać sesję po stronie serwera.
- Podejrzany response nie zawsze jest dowodem; impact trzeba potwierdzić kontrolowanym testem.

## Status końcowy

**PASS — praktyczna podstawa odpowiednia dla Frontend Developera przechodzącego do AppSec.**
