# SQL Injection Login Bypass - Notatki z nauki

> Platforma: PortSwigger Web Security Academy  
> Lab: SQL injection vulnerability allowing login bypass  
> Status: Ukończony  
> Cel: Prywatne notatki AppSec, nie pełny walkthrough.

## TL;DR

Formularz logowania był podatny, ponieważ dane użytkownika najprawdopodobniej były wstawiane bezpośrednio do zapytania SQL. Kontrolując parametr `username` i używając komentarza SQL, można było pominąć sprawdzanie hasła i zalogować się jako `administrator`.

## Co ćwiczyłem

Testowałem formularz logowania pod kątem SQL Injection i sprawdzałem, jak backendowe zapytanie logowania może się zachować, gdy wartość `username` zmienia logikę SQL.

Celem było zalogowanie się jako użytkownik `administrator` bez znajomości hasła.

## Input, który kontrolowałem

Kontrolowanym inputem było pole `username` w requestcie logowania.

Przykładowe body requestu:

```http
POST /login HTTP/2
Content-Type: application/x-www-form-urlencoded

csrf=...&username=administrator%27--&password=qw
```

Odkodowana wartość `username`:

```text
administrator'--
```

## Co się stało

Serwer zwrócił redirect:

```http
HTTP/2 302 Found
Location: /my-account?id=administrator
Set-Cookie: session=...
```

To sugerowało, że login bypass zadziałał, ponieważ aplikacja przekierowała mnie na konto administratora i wystawiła nowe cookie sesyjne.

Po użyciu nowego cookie sesyjnego i wejściu na stronę konta aplikacja pokazała:

```html
Your username is: administrator
```

To potwierdziło, że byłem zalogowany jako administrator.

## Dlaczego to zadziałało

Backendowe zapytanie mogło być zbudowane w podatny sposób, np. podobnie do:

```sql
SELECT * FROM users
WHERE username = 'administrator'
AND password = 'qw';
```

Po wstrzyknięciu w pole username zapytanie mogło wyglądać koncepcyjnie tak:

```sql
SELECT * FROM users
WHERE username = 'administrator'--'
AND password = 'qw';
```

Apostrof zamknął oryginalny string, a sekwencja komentarza SQL zakomentowała resztę zapytania. W efekcie warunek sprawdzający hasło został pominięty.

## Ważna obserwacja: CSRF i stan sesji

Podczas ręcznego testowania w Burp dostałem w pewnym momencie błąd:

```text
Invalid CSRF token (session does not contain a CSRF token)
```

To nie był błąd payloadu SQL Injection. Problem wynikał z niedopasowania tokenu CSRF i sesji.

Token CSRF był powiązany z cookie sesyjnym, więc podczas testowania ręcznego aktualny token CSRF i aktualne cookie sesyjne muszą do siebie pasować.

## Wpływ

W prawdziwej aplikacji taki błąd mógłby pozwolić atakującemu ominąć logowanie i uzyskać dostęp do innego konta, w tym konta administracyjnego.

## Remediacja dla developera

Aplikacja nie powinna budować zapytań SQL przez konkatenację danych kontrolowanych przez użytkownika.

Zalecane poprawki:

- używać parameterized queries / prepared statements;
- traktować username i password jako dane, a nie wykonywalną część SQL;
- utrzymać poprawną obsługę sesji i CSRF;
- używać generycznych komunikatów błędu logowania;
- dodać testy na próby obejścia logowania.

CSRF protection i secure cookies są przydatne, ale nie naprawiają SQL Injection. Główna poprawka to bezpieczne budowanie zapytań.

## Checklist do review

- Czy input użytkownika jest bezpośrednio wstawiany do query SQL?
- Czy używane są prepared statements?
- Czy logikę logowania można zmienić przez dodanie apostrofu lub komentarza?
- Czy testowane są zmodyfikowane requesty, a nie tylko zachowanie UI?
- Czy tokeny CSRF i cookies sesyjne są poprawnie walidowane?

## Najważniejsza lekcja

Komentarze SQL mogą usunąć część podatnego zapytania backendowego. W tym labie zakomentowanie sprawdzania hasła pozwoliło zalogować się jako `administrator`.
