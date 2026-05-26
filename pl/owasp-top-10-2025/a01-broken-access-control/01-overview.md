# A01 Broken Access Control

## Zakres

Ta notatka mapuje moją praktyczną naukę access control na **OWASP Top 10 2025 A01 Broken Access Control**.

Jest napisana z perspektywy Frontend Engineera przechodzącego w stronę AppSec. Fokus nie jest tylko na rozwiązaniu laba, ale na zrozumieniu, jak taki typ problemu wygląda w prawdziwych aplikacjach oraz jak powinien być reviewowany, naprawiany i testowany regresyjnie.

## Co oznacza ta kategoria

Broken Access Control występuje wtedy, gdy aplikacja nie egzekwuje poprawnie, co uwierzytelniony lub nieuwierzytelniony użytkownik może zrobić.

Przydatny podział:

- **Authentication** pyta: kim jesteś?
- **Authorization / Access Control** pyta: czy wolno ci wykonać tę akcję na tym zasobie?

W ukończonym labie authentication działało, bo normalny użytkownik był poprawnie zalogowany. Problemem była authorization: backend nie sprawdzał, czy zalogowany użytkownik ma uprawnienia administratora przed przetworzeniem finalnego requestu zmiany roli.

## Dlaczego to ma znaczenie

Błędy access control mogą pozwolić użytkownikom działać poza ich zamierzonymi uprawnieniami.

W zależności od funkcji może to prowadzić do:

- podglądu danych innego użytkownika,
- zmiany szczegółów konta innego użytkownika,
- dostępu do funkcji tylko dla administratorów,
- modyfikacji ról lub uprawnień,
- pobierania plików spoza zamierzonego zakresu,
- uruchamiania uprzywilejowanej funkcjonalności backendowej,
- dostępu do zasobów wewnętrznych przez funkcje server-side,
- zmiany stanu aplikacji bez poprawnej autoryzacji.

## Abuse case

Użytkownik z niskimi uprawnieniami odkrywa lub rekonstruuje request przeznaczony tylko dla administratora i wysyła go bezpośrednio do backendu.

Ponieważ finalny endpoint zmieniający stan nie egzekwuje autoryzacji, użytkownik podnosi własną rolę do administratora i uzyskuje dostęp do funkcji admin-only.

## Typowe przykłady

Przykłady Broken Access Control:

- normalny użytkownik uzyskuje dostęp do funkcji `/admin`,
- modyfikacja konta innego użytkownika przez zmianę parametru requestu,
- dostęp do faktury, dokumentu lub profilu innego użytkownika przez zmianę ID,
- wykonywanie akcji administracyjnych przez bezpośrednie requesty API,
- brak autoryzacji na finalnym kroku wieloetapowego workflow,
- traktowanie frontend route guardów lub ukrytych przycisków jako głównej ochrony,
- poleganie na poprzednim kroku workflow zamiast autoryzowania endpointu, który wykonuje akcję,
- dostęp do plików spoza zamierzonego katalogu,
- pozwalanie backendowym requestom dotrzeć do zasobów internal-only, gdzie granice dostępu nie są egzekwowane.

## Typowe root causes

Typowe przyczyny:

- brakuje sprawdzeń autoryzacji na jednym lub wielu endpointach backendowych,
- access control jest stosowany tylko w UI,
- access control jest stosowany tylko na początku wieloetapowego procesu,
- backend ufa parametrom requestu takim jak `username`, `userId`, `role` lub `confirmed`,
- brakuje sprawdzeń ownership albo są niespójne,
- funkcje administracyjne są chronione przez obscurity zamiast permission checks,
- logika aplikacji zakłada, że użytkownicy mogą dotrzeć do endpointów tylko przez zamierzony flow UI,
- nie ma centralnej polityki autoryzacji ani reużywalnego guarda,
- testy skupiają się na happy path i nie sprawdzają bezpośrednich requestów od użytkowników z niższymi uprawnieniami.

## Impact

Impact zależy od dotkniętej funkcji.

Dla ukończonego laba impactem było **privilege escalation**:

> Normalny uwierzytelniony użytkownik mógł bezpośrednio wysłać finalny request potwierdzający i podnieść własne konto do administratora.

W prawdziwej aplikacji może to prowadzić do:

- pełnego dostępu do funkcji admin-only,
- nieautoryzowanych zmian ról,
- nadużyć w zarządzaniu użytkownikami,
- nieautoryzowanego dostępu do danych,
- nieautoryzowanej modyfikacji danych,
- utraty zaufania do granic kont i uprawnień,
- możliwego szerszego kompromisu, jeśli funkcje admina udostępniają wrażliwe operacje.

## Powiązane notatki wewnętrzne

Ta kategoria łączy się z kilkoma tematami, które już studiowałem:

- [Broken Access Control / IDOR](../../key-web-vulnerabilities/02-broken-access-control-idor/README.md) - bezpośredni dostęp do obiektu, gdy backend nie sprawdza ownership lub permission.
- [CSRF + SameSite Cookies](../../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md) - powiązane, gdy wrażliwe akcje zmieniające stan polegają tylko na sesji użytkownika.
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md) - powiązane, gdy użytkownicy mogą uzyskać dostęp do plików poza zamierzonym katalogiem lub granicą autoryzacji.
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md) - powiązane, gdy funkcjonalność backendowych requestów może dotrzeć do zasobów wewnętrznych lub uprzywilejowanych lokalizacji sieciowych.
- [Authentication Bypass / Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md) - przydatne tło do oddzielenia authentication failures od authorization failures.

## Mój takeaway jako Frontend Engineer

Kontrolki frontendowe są ważne dla user experience, ale nie są granicą bezpieczeństwa.

React route guard, ukryty link admina, disabled button albo conditional UI check mogą pomóc zapobiegać przypadkowemu użyciu, ale użytkownicy nadal kontrolują przeglądarkę. Mogą wysyłać requesty bezpośrednio przez Burp Suite, Postman, curl, DevTools lub własne skrypty.

Prawdziwy access control musi być egzekwowany na backendowym endpoincie, który wykonuje akcję.

Dobra implementacja frontendowa nadal powinna:

- nie eksponować niepotrzebnego admin UI,
- nie ufać client-side wartościom roli,
- czytelnie obsługiwać odpowiedzi `401` i `403`,
- unikać wyciekania wrażliwej funkcjonalności przez przewidywalne linki,
- czysto wspierać backendowy model access control.

Ale nie może zastąpić backendowej autoryzacji.

## Jak wygląda dobre rozwiązanie

Bezpieczniejsza implementacja powinna:

- egzekwować autoryzację server-side na każdym wrażliwym endpoincie,
- stosować deny-by-default dla chronionych akcji,
- sprawdzać rolę i ownership tam, gdzie jest to potrzebne,
- nie ufać parametrom kontrolowanym przez użytkownika przy decyzjach autoryzacyjnych,
- ponownie sprawdzać uprawnienia na finalnych krokach potwierdzenia,
- centralizować logikę autoryzacji tam, gdzie to możliwe,
- testować bezpośrednie requesty backendowe od użytkowników z niższymi uprawnieniami,
- upewniać się, że stan nie zmienia się po nieudanej autoryzacji,
- logować podejrzane nieautoryzowane próby dla wrażliwych akcji.

## Zewnętrzne referencje

- OWASP Top 10 2025 A01 Broken Access Control: https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/
- PortSwigger Web Security Academy: Access control vulnerabilities: https://portswigger.net/web-security/access-control
- Ukończony lab PortSwigger: Multi-step process with no access control on one step: https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step
