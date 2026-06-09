# Checklista Review Authentication

Użyj jej podczas review frontendu, API, middleware, CMS lub procesu AppSec.

## Tożsamość i flow

- [ ] Zidentyfikuj wszystkie wejścia: login, registration, verification, reset, recovery, MFA, remember-me, fallback SSO i support-assisted recovery.
- [ ] Zapisz identity claim używany w każdym flow.
- [ ] Zapisz oczekiwany dowód uwierzytelnienia.
- [ ] Wskaż komponent backendu, który weryfikuje dowód.
- [ ] Potwierdź, że kroków nie można pominąć, przestawić, replayować ani wywołać bezpośrednio.
- [ ] Potwierdź spójne zasady na alternatywnych endpointach.

## Logowanie hasłem

- [ ] Istniejący i nieistniejący użytkownik otrzymują równoważne błędy po stronie klienta.
- [ ] Status, długość response i timing nie tworzą niezawodnej enumeracji.
- [ ] Nieudane próby są ograniczane warstwowo, a nie tylko jednym słabym sygnałem.
- [ ] Account lock nie pozwala na łatwy denial of service.
- [ ] Znane słabe lub wyciekłe hasła są odrzucane zgodnie z polityką produktu.
- [ ] Default i hard-coded credentials nie trafiają na środowisko.
- [ ] Błędy logowania są rejestrowane bez plaintext passwords.

## Password Reset i Recovery

- [ ] Odpowiedzi resetu nie ujawniają istnienia konta.
- [ ] Reset token jest wystarczająco nieprzewidywalny.
- [ ] Token wygasa w odpowiednim czasie.
- [ ] Token jest jednorazowy.
- [ ] Token jest powiązany z jednym kontem i jednym celem.
- [ ] Konto docelowe wynika z zaufanego stanu tokenu.
- [ ] Kontrolowany `username`, email lub user ID nie może zmienić konta docelowego.
- [ ] Link resetujący używa zaufanego origin HTTPS.
- [ ] Host/proxy headers nie pozwalają zatruć URL resetu.
- [ ] Udany reset natychmiast unieważnia token.
- [ ] Istniejące sesje są unieważniane lub obsługiwane zgodnie z udokumentowaną decyzją ryzyka.
- [ ] Reset hasła nie wyłącza ani nie omija MFA bez dodatkowej kontroli.
- [ ] Recovery i support-assisted paths są co najmniej tak silne jak normalny login.

## MFA

- [ ] Aplikacja rozróżnia `mfa_pending` i `mfa_completed`.
- [ ] Samo hasło nie tworzy fully authenticated session.
- [ ] Przed MFA sesja może używać tylko endpointów potrzebnych do zakończenia authentication.
- [ ] Każda chroniona trasa i API sprawdza ukończenie MFA po stronie serwera.
- [ ] Bezpośrednia nawigacja nie omija MFA.
- [ ] Kody MFA wygasają.
- [ ] Próby MFA są rate-limited.
- [ ] Kod jest powiązany z poprawnym użytkownikiem i próbą.
- [ ] Backup codes są ograniczone, chronione i unieważniane po użyciu.
- [ ] Wyłączenie MFA wymaga recent authentication.
- [ ] Trusted-device ma jawny lifetime i revocation.

## Session Management

- [ ] Po loginie wydawany jest nowy, nieprzewidywalny session ID.
- [ ] Identyfikator rotuje po zmianie uprawnień lub step-up authentication, gdy jest to potrzebne.
- [ ] Sesja anonimowa/pre-auth nie jest promowana bez rotacji.
- [ ] Logout unieważnia sesję po stronie serwera.
- [ ] Stara sesja nie działa po logout.
- [ ] Skonfigurowano idle i absolute timeout.
- [ ] Remember-me ma odpowiedni lifetime, rotację i revocation.
- [ ] Reset hasła, zmiana hasła, wyłączenie konta i podejrzenie przejęcia mają zdefiniowane zasady revocation.
- [ ] Concurrent sessions odpowiadają wymaganiom produktu.
- [ ] Identyfikatory sesji nie pojawiają się w URL ani logach.

## Cookies i stan przeglądarki

- [ ] Session cookie używa `HttpOnly`, gdy JavaScript go nie potrzebuje.
- [ ] Session cookie używa `Secure`.
- [ ] `SameSite` odpowiada flow aplikacji i ochronie CSRF.
- [ ] `Path` i `Domain` są możliwie wąskie.
- [ ] Authentication nie zależy od edytowalnych wartości w localStorage/sessionStorage.
- [ ] Wrażliwe tokeny nie są bez potrzeby utrwalane w browser storage.
- [ ] Frontend czyści stan po logout, ale backend invalidation pozostaje podstawową kontrolą.

## Wrażliwe akcje

- [ ] Zmiana hasła wymaga bieżącego hasła lub recent reauthentication, gdy to uzasadnione.
- [ ] Zmiana emaila, telefonu, MFA, płatności lub recovery wymaga step-up authentication przy odpowiednim ryzyku.
- [ ] Wrażliwe zmiany generują powiadomienie użytkownika.
- [ ] Sama istniejąca sesja nie zawsze jest wystarczającym dowodem dla akcji wysokiego ryzyka.

## Logging i Detection

- [ ] Udane i nieudane authentication events są rejestrowane z użytecznym kontekstem.
- [ ] Reset requests, nieprawidłowe tokeny i token replay są logowane.
- [ ] MFA failures i nietypowe retry mogą wywołać alert.
- [ ] Monitorowane są credential stuffing i password spraying.
- [ ] Session creation, revocation i podejrzany reuse są widoczne.
- [ ] Logi nie zawierają haseł, pełnych reset tokenów ani zbędnych sekretów.

## Frontend Review

- [ ] Route guards są traktowane jako UX, nie security enforcement.
- [ ] API niezależnie od UI odrzuca sesje unauthenticated i MFA-pending.
- [ ] Disabled controls nie zastępują server-side rate limiting ani authorization.
- [ ] Client validation jest powtórzona i egzekwowana na backendzie.
- [ ] UI nie ujawnia istnienia kont przez zbędne różnice komunikatów.
