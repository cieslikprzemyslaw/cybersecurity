# PortSwigger Lab 02 - Upload web shella przez bypass ograniczenia Content-Type

## Co testowałem

Testowałem, czy walidacja uploadu ufa nagłówkowi `Content-Type` konkretnej części pliku.

## Co znalazłem

Aplikacja odrzucała plik PHP, gdy został wysłany jako:

```http
Content-Type: application/octet-stream
```

Odpowiedź zawierała komunikat:

```text
Only image/jpeg and image/png are allowed
```

Ten sam plik PHP został zaakceptowany po zmianie `Content-Type` części pliku na:

```http
Content-Type: image/png
```

Nazwa pliku pozostała taka sama:

```text
shell.php
```

Serwer później wykonał uploadowany plik PHP.

## Przyczyna źródłowa

Aplikacja ufała kontrolowanemu przez użytkownika nagłówkowi multipart jako głównemu mechanizmowi walidacji.

## Wpływ

Atakujący mógł wgrać PHP web shell podszywający się pod obraz i wykonywać komendy na serwerze.

## Pomysł na test regresji

Upload `shell.php` z `Content-Type: image/png` nie powinien skutkować zapisaniem ani udostępnieniem wykonywalnego pliku.

## Główna lekcja

`Content-Type` w requestcie multipart uploadu jest wskazówką, a nie dowodem.
