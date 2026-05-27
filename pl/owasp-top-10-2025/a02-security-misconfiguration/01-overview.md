# A02 Security Misconfiguration

## Zakres

Ta notatka mapuje moją praktyczną naukę na **OWASP Top 10 2025 A02 Security Misconfiguration**.

Jest napisana z perspektywy Frontend Engineera przechodzącego w stronę AppSec. Fokus nie jest tylko na rozwiązaniu laba, ale na rozpoznawaniu niebezpiecznego zachowania produkcyjnego, review symptomów konfiguracji z poziomu przeglądarki/API oraz opisaniu, jak problem powinien być naprawiony i testowany regresyjnie.

## Co oznacza ta kategoria

Security Misconfiguration występuje wtedy, gdy aplikacja, API, framework, serwer, kontener, usługa chmurowa, CMS, reverse proxy albo środowisko deploymentu są skonfigurowane w niebezpieczny sposób.

Kod aplikacji może wyglądać poprawnie, ale konfiguracja lub zachowanie runtime nadal mogą ujawniać:

- wrażliwe informacje,
- niepotrzebną funkcjonalność,
- słabe wartości domyślne,
- funkcje debugowe,
- niebezpieczną obsługę błędów,
- nieutwardzone zachowanie platformy.

W prostych słowach:

> Aplikacja jest podatna, bo coś wokół niej zostało zbyt otwarte, zbyt gadatliwe, zbyt permisywne albo zbyt podobne do konfiguracji developerskiej.

## Dlaczego to ma znaczenie

Security Misconfiguration często daje atakującym informacje lub dostęp, których nie powinni mieć.

Błędna konfiguracja może ujawniać:

- strony debugowe,
- stack trace'y,
- wewnętrzne ścieżki plików,
- wersje frameworka lub serwera,
- zmienne środowiskowe,
- klucze API lub tokeny,
- pliki backupów,
- endpointy developerskie,
- domyślne dane logowania,
- zbyt permisywną konfigurację CORS,
- niepotrzebne usługi lub route'y,
- brakujące nagłówki hardeningowe.

Dla atakującego zmniejsza to liczbę zgadywanek. Verbose error albo wystawiona strona debugowa mogą zdradzić, jak aplikacja jest zbudowana, gdzie znajdują się pliki, jaki framework jest używany i które części aplikacji warto atakować dalej.

## Abuse case

Użytkownik wysyła nieoczekiwane dane wejściowe do publicznego endpointu API.

Zamiast zwrócić kontrolowany błąd dla klienta, aplikacja zwraca informacje debugowe, w tym traceback, wewnętrzną ścieżkę pliku, nazwę funkcji, numer linii i wrażliwą wartość.

Atakujący używa tych informacji do rozpoznania aplikacji i planowania kolejnych ataków na stack.

## Praktyczne wzorce z tego sprintu

W tym temacie przećwiczyłem dwa typowe wzorce A02.

### Wzorzec 1: Verbose error / debug response

Lab TryHackMe wystawiał User Management API:

```http
GET /api/user/<user_id>
GET /api/user/123
```

Dokumentacja API mówiła, że user ID musi być numeryczne.

Normalna wartość numeryczna zwracała generyczny obiekt użytkownika:

```json
{
  "email": "john@example.com",
  "id": "9999",
  "name": "John Doe"
}
```

Jednak po podaniu wartości nienumerycznej, na przykład `admin`, API zwracało verbose debug object zamiast kontrolowanego błędu. Odpowiedź ujawniała traceback, wewnętrzną ścieżkę pliku, nazwę funkcji, numer linii, typ wyjątku i wrażliwą wartość labową.

To jest Security Misconfiguration, bo wewnętrzne informacje debugowe i wrażliwe dane pojawiły się w odpowiedzi widocznej dla klienta. Pełne dowody z laba są w [02-labs-or-practice.md](02-labs-or-practice.md), a wersja raportowa dowodów jest w [security-findings/01-example-finding.md](security-findings/01-example-finding.md).

### Wzorzec 2: Publiczny debug endpoint / `phpinfo()`

Lab PortSwigger pokazał debug endpoint odkrywalny z poziomu HTML source:

```html
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

Link nie był widoczny w UI, ale nadal został wysłany do przeglądarki. Bezpośrednie wejście na endpoint:

```http
GET /cgi-bin/phpinfo.php
```

zwracało publiczną stronę `phpinfo()`.

Strona ujawniała szczegóły konfiguracji serwera i PHP, między innymi wersję PHP, system operacyjny, ścieżki do plików konfiguracyjnych, załadowane moduły, ustawienia środowiska oraz wrażliwy `SECRET_KEY`.

To również jest Security Misconfiguration, bo funkcjonalność diagnostyczna/developerska była dostępna publicznie. Wersja raportowa jest w [security-findings/02-public-debug-endpoint-phpinfo.md](security-findings/02-public-debug-endpoint-phpinfo.md).

## Typowe przykłady

Przykłady Security Misconfiguration:

- debug mode włączony na produkcji,
- verbose errors zwracane użytkownikom,
- stack trace'y ujawniane w odpowiedziach API,
- publiczne debug endpointy takie jak `phpinfo()`,
- publiczne `/debug`, `/config`, `/status`, `/server-status`, `/actuator` lub `/health`,
- domyślne konta lub hasła zostawione aktywne,
- niepotrzebne route'y, usługi, pluginy albo strony admina wystawione publicznie,
- endpointy testowe lub stagingowe dostępne z internetu,
- brakujące lub słabe nagłówki bezpieczeństwa,
- zbyt permisywna konfiguracja CORS,
- włączone directory listing,
- wystawione pliki konfiguracyjne takie jak `.env`, backupy, logi albo stare pliki deploymentu,
- publiczne source mapy bez zaakceptowanego powodu,
- cloud storage bucket lub usługi ze zbyt szerokimi uprawnieniami,
- niespójna konfiguracja między środowiskami.

## Typowe root causes

Typowe przyczyny:

- ustawienia developerskie przypadkowo wdrożone na produkcję,
- brak globalnej obsługi błędów,
- komunikaty wyjątków zawierające wrażliwe dane,
- sekrety umieszczone w error messages lub logach,
- brak konfiguracji specyficznej dla środowiska,
- brak checklisty hardeningowej dla deploymentu,
- brak automatycznych sprawdzeń nagłówków bezpieczeństwa lub wystawionych plików,
- domyślne zachowanie frameworka pozostawione bez review,
- debug files przypadkowo wdrożone na produkcję,
- poleganie na ukrytych linkach zamiast na usunięciu lub ochronie endpointu,
- niejasna odpowiedzialność za konfigurację bezpieczeństwa,
- configuration drift między środowiskami.

## Impact

Impact zależy od tego, co zostało ujawnione.

W tym labie aplikacja ujawniła flagę i wewnętrzny traceback. W prawdziwej aplikacji podobne zachowanie mogłoby ujawnić:

- klucze API,
- JWT signing secrets,
- connection stringi do bazy danych,
- nazwy użytkowników lub adresy email,
- wewnętrzne ścieżki,
- szczegóły frameworka,
- lokalizacje kodu źródłowego,
- zapytania SQL,
- stack trace'y,
- cloud metadata,
- nazwy usług,
- szczegóły wewnętrznej architektury.

Takie informacje mogą wspierać dalsze ataki, na przykład path traversal, LFI, injection, SSRF, credential attacks albo ukierunkowaną eksploatację znanego frameworka/komponentu.

Jeśli zostanie ujawniony `SECRET_KEY` lub podobny sekret aplikacyjny, należy traktować go jako skompromitowany. W zależności od sposobu użycia sekretu atakujący może próbować manipulować podpisanymi cookies, sesjami, tokenami CSRF, JWT, reset tokenami lub innymi danymi zaufanymi przez aplikację. Taki sekret powinien zostać obrócony.

## Powiązane notatki wewnętrzne

Ta kategoria łączy się z kilkoma tematami, które już studiowałem:

- [HTTP, Request/Response i podstawy Auth](../../fundamentals/01-http-request-response-auth.md) - przydatne do rozpoznawania niebezpiecznych statusów, nagłówków i response bodies.
- [Burp Suite Proxy i Repeater](../../fundamentals/02-burp-suite-proxy-repeater.md) - przydatne do porównywania normalnych odpowiedzi z error responses.
- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md) - przydatne do znajdowania wystawionych ścieżek debug, config, backup lub admin.
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md) - powiązane, gdy ujawnione ścieżki lub pliki zdradzają filesystem.
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md) - powiązane, gdy konfiguracja wystawia usługi wewnętrzne lub metadata endpoints.
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md) - powiązane, gdy konfiguracja serwera pozwala uploadowanym plikom wykonywać się lub być serwowanym w niebezpieczny sposób.

## Mój takeaway jako Frontend Engineer

Jako Frontend Engineer nie zawsze konfiguruję backend, serwer lub infrastrukturę bezpośrednio, ale nadal mogę rozpoznawać symptomy misconfiguration z poziomu przeglądarki, DevTools, odpowiedzi API i Burp Suite.

Ważne obserwacje frontend/AppSec:

- Odpowiedzi API nie powinny ujawniać stack trace'ów ani wewnętrznych ścieżek plików.
- Komunikaty błędów w UI powinny być przyjazne dla użytkownika, a nie debugowe.
- Aplikacje frontendowe nie powinny polegać na sekretach osadzonych w kodzie klienta.
- Buildy produkcyjne nie powinny ujawniać source map, debug flags ani funkcji development-only bez jasnego zaakceptowanego powodu.
- Client-side error handling nie powinien wyświetlać surowych wyjątków backendowych.
- Nagłówki bezpieczeństwa i zachowanie CORS często da się zreviewować z przeglądarki.
- Endpointy debugowe i wewnętrzne API nie powinny być odkrywalne z publicznych assetów frontendowych.
- Ukrycie linku w HTML comment nie chroni backendowego endpointu.

Kluczowa lekcja:

> Systemy produkcyjne powinny failować bezpiecznie. Niepoprawny input powinien dawać kontrolowaną, generyczną odpowiedź, a szczegółowe informacje debugowe powinny zostać w logach server-side.

## Jak wygląda dobre rozwiązanie

Bezpieczniejsza implementacja powinna:

- wyłączać debug mode na produkcji,
- używać konfiguracji specyficznej dla produkcji,
- walidować input przed uruchomieniem logiki biznesowej,
- zwracać generyczne komunikaty błędów dla klienta,
- logować szczegółowe błędy techniczne tylko server-side,
- trzymać sekrety poza exception messages i response bodies,
- usuwać debug files z produkcyjnych deploymentów,
- blokować publiczny dostęp do endpointów diagnostycznych,
- obracać sekrety, które zostały ujawnione,
- standaryzować formaty error responses API,
- usuwać niepotrzebne usługi, route'y, pluginy i strony testowe,
- chronić endpointy admin, debug, config i status,
- hardenować nagłówki i zachowanie CORS,
- dodawać testy regresji, które failują, jeśli informacje debugowe pojawią się w odpowiedziach.

## Zewnętrzne referencje

- OWASP Top 10 2025 A02 Security Misconfiguration: https://owasp.org/Top10/2025/A02_2025-Security_Misconfiguration/
- OWASP Cheat Sheet Series: Error Handling: https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html
- OWASP Cheat Sheet Series: HTTP Headers: https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html
