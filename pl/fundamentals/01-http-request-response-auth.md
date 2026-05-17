# HTTP, Request/Response i podstawy Auth

## 1. Overview

Ta notatka opisuje podstawy komunikacji aplikacji webowej oraz miejsca, w których powinny być podejmowane decyzje bezpieczeństwa.

Zanim zacznę testować podatności takie jak IDOR, SQL Injection, XSS, CSRF albo Broken Access Control, muszę dobrze rozumieć, jak działają requesty i response'y. Większość testowania bezpieczeństwa aplikacji webowych zaczyna się od prostego pytania:

> Co klient wysyła do serwera i jak serwer na to odpowiada?

Z perspektywy AppSec przeglądarka jest tylko jednym ze sposobów wysyłania requestów. Ten sam request można często wysłać lub zmodyfikować za pomocą narzędzi takich jak Burp Suite, Postman, curl, DevTools albo własny skrypt.

## 2. HTTP Request

HTTP request to wiadomość wysyłana przez klienta do serwera.

Klientem może być:

- przeglądarka,
- aplikacja mobilna,
- Postman,
- Burp Suite,
- curl,
- inny backend,
- własny skrypt.

Przykładowy request:

```http
GET /products?id=123 HTTP/1.1
Host: example.com
Cookie: session=abc123
User-Agent: Chrome
```

Request powstaje, gdy użytkownik:

- otwiera URL,
- klika link,
- wysyła formularz,
- wysyła dane do API,
- uploaduje plik,
- loguje się,
- zmienia dane konta,
- wykonuje akcję w aplikacji.

W AppSec nie powinienem patrzeć tylko na to, co pokazuje UI. Powinienem sprawdzić, jaki request faktycznie jest wysyłany.

## 3. Popularne metody HTTP

Metody HTTP opisują intencję requestu.

| Method | Typowe zastosowanie |
|---|---|
| `GET` | Pobieranie danych |
| `POST` | Wysyłanie lub tworzenie danych |
| `PUT` | Zastąpienie albo aktualizacja zasobu |
| `PATCH` | Częściowa aktualizacja zasobu |
| `DELETE` | Usunięcie zasobu |
| `OPTIONS` | Sprawdzenie, jakie metody są dozwolone |
| `HEAD` | Podobne do GET, ale bez body odpowiedzi |

Przykłady:

```http
GET /api/users/123
POST /api/users
PUT /api/users/123
PATCH /api/users/123
DELETE /api/users/123
```

Z perspektywy bezpieczeństwa każda metoda ma znaczenie. Ukryty lub niezabezpieczony endpoint `PUT`, `PATCH` albo `DELETE` może być groźniejszy niż zwykły `GET`, bo może zmieniać albo usuwać dane.

## 4. HTTP Response

HTTP response to odpowiedź serwera na request.

Response zwykle zawiera:

- status code,
- headers,
- response body,
- cookies ustawiane przez serwer,
- komunikaty błędów,
- redirecty,
- JSON, HTML, pliki lub inne dane.

Przykładowy response:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Set-Cookie: session=abc123; HttpOnly; Secure
```

```json
{
  "id": 123,
  "name": "Product"
}
```

Popularne status codes:

| Status code | Znaczenie |
|---|---|
| `200 OK` | Request zakończony sukcesem |
| `201 Created` | Zasób został utworzony |
| `302 Found` | Redirect |
| `400 Bad Request` | Niepoprawny request |
| `401 Unauthorized` | Użytkownik nie jest uwierzytelniony |
| `403 Forbidden` | Użytkownik jest zalogowany, ale nie ma uprawnień |
| `404 Not Found` | Zasób nie został znaleziony |
| `405 Method Not Allowed` | Metoda HTTP nie jest dozwolona |
| `500 Internal Server Error` | Błąd po stronie serwera |

Podczas testowania bezpieczeństwa status code jest ważny, ale nie wystarczy. Trzeba też sprawdzać response body, bo czasem wrażliwe dane są zwracane nawet wtedy, gdy UI ich nie pokazuje.

## 5. Co kontroluje użytkownik?

Z perspektywy AppSec powinienem zakładać, że użytkownik może zmienić wszystko, co jest wysyłane w requestcie.

Elementy kontrolowane przez użytkownika mogą obejmować:

- metodę HTTP: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`,
- ścieżkę URL,
- query parameters,
- request body,
- headers,
- cookies,
- tokeny,
- wartości formularzy,
- ukryte pola formularzy,
- ID zasobów,
- uploadowane pliki,
- content type.

Przykład:

```http
GET /profile?id=123
```

Użytkownik może spróbować zmienić to na:

```http
GET /profile?id=124
```

Albo zmienić metodę:

```http
DELETE /profile/123
```

Najważniejsza zasada:

> Never trust user input.

Nawet jeśli request pochodzi z normalnej przeglądarki, nadal może zostać zmodyfikowany zanim dotrze do serwera.

## 6. Co musi kontrolować serwer?

Serwer powinien odpowiadać za decyzje bezpieczeństwa.

Serwer musi kontrolować:

- authentication,
- authorization,
- walidację sesji,
- dostęp do zasobów,
- walidację po stronie serwera,
- dane zwracane użytkownikowi,
- dozwolone akcje,
- obsługę błędów,
- tworzenie cookies,
- zapytania do bazy danych,
- sprawdzanie ról i uprawnień.

Na przykład, jeśli request zawiera:

```json
{
  "role": "admin"
}
```

serwer nie powinien ufać tej wartości. Powinien sprawdzić prawdziwą rolę użytkownika z zaufanego źródła, np. z sesji albo bazy danych.

## 7. GET vs POST - błędne założenie o bezpieczeństwie

Częsty mit mówi, że `POST` jest bezpieczniejszy niż `GET`.

To nieprawda.

`GET` zwykle pobiera dane:

```http
GET /product?id=1
```

`POST` zwykle wysyła dane:

```http
POST /login
```

Ale bezpieczeństwo nie wynika z samej metody HTTP.

Zarówno `GET`, jak i `POST` mogą być podatne na:

- SQL Injection,
- XSS,
- IDOR,
- Broken Access Control,
- CSRF,
- Command Injection,
- wyciek wrażliwych danych.

Także metody takie jak `PUT`, `PATCH` i `DELETE` mogą wprowadzać poważne ryzyko, jeśli brakuje kontroli dostępu.

Przykład:

```http
DELETE /api/users/124
```

Jeśli zwykły użytkownik może usunąć konto innego użytkownika, problemem nie jest sama metoda HTTP. Problemem jest brak autoryzacji po stronie serwera.

## 8. Authentication vs Authorization

Authentication i authorization to dwa różne pojęcia.

### Authentication

Authentication oznacza potwierdzenie tożsamości.

Pytanie brzmi:

> Kim jesteś?

Przykłady authentication:

- login i hasło,
- MFA,
- session cookie,
- JWT,
- API token,
- magic link.

### Authorization

Authorization oznacza sprawdzenie uprawnień.

Pytanie brzmi:

> Co wolno Ci zrobić?

Przykład:

```http
GET /profile/123
```

Zmienione na:

```http
GET /profile/124
```

Jeśli użytkownik A może zobaczyć profil użytkownika B po zmianie ID, oznacza to, że jest zalogowany, ale nie ma poprawnie sprawdzonych uprawnień.

To jest problem Broken Access Control i może być przykładem IDOR.

## 9. Cookies i sesje

Cookie sesyjne często przechowuje identyfikator sesji.

Przykład:

```http
Cookie: sessionId=abc123
```

Serwer musi sprawdzić:

- czy sesja istnieje,
- czy sesja jest poprawna,
- czy sesja nie wygasła,
- do którego użytkownika należy sesja,
- jakie uprawnienia ma użytkownik.

Serwer nie powinien ślepo ufać cookies, ponieważ użytkownik może próbować:

- je zmienić,
- skopiować,
- usunąć,
- podmienić,
- użyć starych wartości,
- wysłać fałszywe wartości.

Przykład:

```http
Cookie: sessionId=abc123
```

Zmienione na:

```http
Cookie: sessionId=test
```

Bezpieczna aplikacja powinna odrzucić niepoprawne lub wygasłe wartości sesji.

## 10. Key Takeaways

- HTTP request to wiadomość wysyłana przez klienta do serwera.
- HTTP response to odpowiedź serwera.
- Użytkownik może zmieniać metody, ścieżki, parametry, body, headers, cookies i tokeny.
- `GET`, `POST`, `PUT`, `PATCH` i `DELETE` mogą mieć znaczenie bezpieczeństwa.
- `POST` nie jest automatycznie bezpieczniejszy niż `GET`.
- Authentication oznacza potwierdzenie tożsamości.
- Authorization oznacza sprawdzenie uprawnień.
- Zalogowany użytkownik nie oznacza automatycznie użytkownika uprawnionego do każdego zasobu.
- Decyzje bezpieczeństwa muszą być egzekwowane po stronie serwera.
- Nie należy ufać danym kontrolowanym przez użytkownika.

## 11. Practical AppSec Checklist

Patrząc na request, powinienem zadać sobie pytania:

- Jaka metoda HTTP jest używana?
- Czy metodę można zmienić?
- Jaka ścieżka jest requestowana?
- Czy w URL, query string albo body są jakieś ID?
- Czy użytkownik ma prawo do tego zasobu?
- Czy użytkownik ma prawo wykonać tę akcję?
- Czy serwer waliduje input?
- Czy response ujawnia więcej danych niż powinien?
- Czy cookies albo tokeny są używane poprawnie?
- Co się stanie, jeśli zmienię ID, metodę, cookie albo body?

Główny mindset:

> Nie pytaj tylko, czy request działa. Zapytaj, czy użytkownik powinien mieć prawo go wykonać.
