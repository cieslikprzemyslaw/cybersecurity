# A01 Laby i praktyka: Broken Access Control

## Ukończona praktyka

### PortSwigger Web Security Academy

**Lab:** Multi-step process with no access control on one step
**Kategoria:** Access control vulnerabilities and privilege escalation
**Status:** Ukończony
**Główny wzorzec:** Brak autoryzacji na finalnym kroku wieloetapowego workflow administracyjnego

Link zewnętrzny:

- https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step

## Kontekst laba

To był celowo podatny lab PortSwigger użyty do legalnej nauki i dokumentacji.

Lab zawierał panel admina z wieloetapowym procesem zmiany roli użytkownika. Pierwszy krok workflow podnoszenia roli był chroniony, ale finalny request potwierdzający nie był poprawnie chroniony.

Celem było zrozumienie workflow admina, a potem sprawdzenie, czy normalny uwierzytelniony użytkownik może bezpośrednio wysłać finalny request potwierdzający.

## Co ćwiczyłem

W tym labie ćwiczyłem:

- obserwowanie workflow admin-only w Burp Suite,
- oddzielanie widocznego flow UI od faktycznych requestów backendowych,
- identyfikowanie requestu, który wykonuje realną akcję zmiany stanu,
- testowanie requestów z sesją normalnego użytkownika,
- porównanie autoryzacji pierwszego kroku z autoryzacją finalnego kroku,
- zrozumienie, dlaczego każdy wrażliwy endpoint backendowy musi mieć własny access-control check,
- użycie wyniku laba jako evidence dla znaleziska w stylu AppSec.

## Ważne rozróżnienie

Problemem nie było to, że użytkownik mógł zalogować się jako administrator.

Problemem było to, że normalny uwierzytelniony użytkownik mógł bezpośrednio wysłać finalny request potwierdzający dla uprzywilejowanej akcji.

To czyni podatność błędem authorization/access control, a nie authentication failure.

## Zaobserwowany workflow

Proces podnoszenia roli przez admina miał dwa ważne requesty.

### Krok 1: Początkowy request podniesienia roli

Normalny użytkownik próbował wysłać pierwszy request:

```http
POST /admin-roles
Cookie: session=<normal-user-session>
Content-Type: application/x-www-form-urlencoded

username=wiener&action=upgrade
```

Serwer odrzucił request:

```http
HTTP/2 401 Unauthorized
```

To pokazało, że aplikacja miała pewien access control na pierwszym kroku workflow.

### Krok 2: Finalny request potwierdzający

Normalny użytkownik wysłał potem finalny request potwierdzający bezpośrednio:

```http
POST /admin-roles
Cookie: session=<normal-user-session>
Content-Type: application/x-www-form-urlencoded

action=upgrade&confirmed=true&username=wiener
```

Serwer zaakceptował request i przekierował użytkownika do obszaru admina:

```http
HTTP/2 302 Found
Location: /admin
```

To wskazywało, że finalny krok potwierdzenia wykonywał uprzywilejowaną akcję bez sprawdzenia, czy aktualny użytkownik ma uprawnienia administratora.

## Co było podatne

Podatny był finalny request potwierdzający wysłany z sesją normalnego użytkownika:

```http
POST /admin-roles
Cookie: session=<normal-user-session>
Content-Type: application/x-www-form-urlencoded

action=upgrade&confirmed=true&username=wiener
```

Endpoint wyglądał, jakby ufał stanowi workflow albo parametrowi `confirmed=true` zamiast autoryzować użytkownika w miejscu, w którym wykonywana była zmiana roli.

## Dlaczego to jest Broken Access Control

To jest Broken Access Control, ponieważ aplikacja nie sprawdziła, czy uwierzytelniony użytkownik może wykonać żądaną akcję.

Użytkownik był uwierzytelniony jako `wiener`, ale `wiener` nie powinien być autoryzowany do zmiany ról.

Pierwszy krok był chroniony, ale finalny krok zmieniający stan nie był. To pozwoliło normalnemu użytkownikowi ominąć chroniony krok i bezpośrednio wykonać uprzywilejowaną akcję.

## Co mnie zmyliło albo było warte zauważenia

### Obsługa session cookie

Test zadziałał dopiero po użyciu aktualnego session cookie normalnego użytkownika.

To było przydatne przypomnienie, że Repeater może łatwo zawierać stare cookies z innego flow logowania. Przy testowaniu access control sesja musi pasować do roli użytkownika, którą testujemy.

### Pierwszy request vs finalny request

Pierwszy request zwrócił `401 Unauthorized`, przez co początkowo wyglądało, że flow jest chroniony.

Ważna lekcja: access control trzeba testować na każdym requeście, szczególnie na requeście, który faktycznie zmienia stan.

### `401` vs `403`

W labie odpowiedź dla chronionego requestu to `401 Unauthorized`.

W prawdziwej aplikacji, jeśli użytkownik jest uwierzytelniony, ale nie ma uprawnień, `403 Forbidden` jest zwykle czytelniejszą odpowiedzią. `401` zwykle oznacza brak lub nieważną authentication, a `403` oznacza, że użytkownik jest uwierzytelniony, ale nie ma dostępu.

### UI vs backend authorization

Normalny użytkownik nie potrzebował admin UI. Request można było wysłać bezpośrednio przez Burp Suite.

To wzmacnia lekcję, że ukrycie admin UI nie wystarcza. Backend authorization musi być egzekwowane niezależnie.

## Jak testowałbym to w prawdziwej aplikacji

Szukałbym tego wzorca w:

- workflow zarządzania rolami,
- administracyjnych akcjach zarządzania użytkownikami,
- approval i moderation flows,
- workflow publikowania,
- flow usuwania konta,
- krokach potwierdzenia zmiany emaila lub hasła,
- zmianach płatności lub billingowych,
- każdym wieloetapowym workflow, gdzie finalny krok zmienia stan.

Dla każdego workflow testowałbym:

- czy użytkownik z niższymi uprawnieniami może wywołać każdy krok bezpośrednio,
- czy finalny endpoint potwierdzający sprawdza uprawnienia,
- czy zmiana parametrów takich jak `username`, `userId`, `role`, `action` lub `confirmed` zmienia wynik,
- czy backend zwraca `403 Forbidden` dla uwierzytelnionych, ale nieautoryzowanych użytkowników,
- czy underlying state pozostaje bez zmian po nieautoryzowanym requeście.

## Wynik review

Praktyczna lekcja z tego laba jest prosta: aplikacja sprawdzała użytkownika na początku workflow admina, ale zapomniała sprawdzić go ponownie w momencie, w którym wykonywana była prawdziwa zmiana.

Z nietechnicznej perspektywy to trochę jak sprawdzenie komuś identyfikatora przy wejściu, a potem pozwolenie tej osobie użyć panelu tylko dla pracowników bez sprawdzenia, czy faktycznie jest pracownikiem.

Główne wnioski:

- użytkownik był zalogowany, ale nie powinien móc wykonać akcji administratora,
- finalny request potwierdzający był najważniejszy, bo to on zmieniał rolę konta,
- backend powinien sprawdzać uprawnienia dokładnie w miejscu, w którym wykonywana jest zmiana,
- ukrycie ekranu admina w UI nie rozwiązałoby problemu, bo request nadal można wysłać bezpośrednio,
- dobra poprawka potrzebuje testów, które potwierdzają zarówno błąd odpowiedzi, jak i to, że rola użytkownika się nie zmieniła.

## Powiązane notatki wewnętrzne

- [Broken Access Control / IDOR](../../key-web-vulnerabilities/02-broken-access-control-idor/README.md)
- [Authentication Bypass / Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [CSRF + SameSite Cookies](../../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md)
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)

## Pomysły na przyszłą praktykę

Możliwa następna praktyka związana z A01 obejmuje:

- PortSwigger: Method-based access control can be circumvented
- PortSwigger: URL-based access control can be circumvented
- Zadanie praktyczne: przejrzeć przykładową mapę tras API i wskazać, które endpointy wymagają sprawdzeń roli i ownership
- Zadanie praktyczne: napisać pomysły na testy regresji dla workflow admin-only
