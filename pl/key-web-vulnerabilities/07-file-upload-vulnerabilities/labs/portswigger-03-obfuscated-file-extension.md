# PortSwigger Lab 03 - Upload web shella przez zaciemnione rozszerzenie pliku

## Co testowałem

Testowałem, jak aplikacja obsługuje walidację nazwy pliku i rozszerzenia.

## Co znalazłem

```text
shell.php                  -> odrzucony
shell.php + image/png      -> odrzucony
shell.php.jpg              -> zaakceptowany, ale serwowany jako obraz
shell.jpg.php              -> odrzucony
shell.php%00.jpg           -> zaakceptowany i zapisany jako shell.php
shell.php?cmd=...          -> wykonany po skutecznym zapisaniu jako .php
```

## Dlaczego `shell.php.jpg` nie wystarczył

Upload został zaakceptowany, ale finalny plik był serwowany jako obraz:

```http
Content-Type: image/jpeg
```

Kod PHP pojawił się w body odpowiedzi jako tekst, co pokazało, że serwer go nie wykonał.

## Dlaczego `shell.php%00.jpg` zadziałał

Walidacja zaakceptowała nazwę pliku, ponieważ wyglądała tak, jakby kończyła się dozwolonym rozszerzeniem obrazu.

Aplikacja zapisała jednak plik jako:

```text
shell.php
```

Uploadowany plik został następnie wykonany przez serwer jako PHP.

## Przyczyna źródłowa

Słaba walidacja nazwy pliku i rozszerzenia oraz niebezpieczna obsługa znaków zakodowanych lub znaków null byte w nazwie pliku.

## Pomysł na test regresji

Aplikacja powinna odrzucać albo bezpiecznie obsługiwać:

```text
shell.php
shell.php.jpg
shell.jpg.php
shell.php%00.jpg
shell.php%00.png
shell.PHP
shell.pHp
shell%2ephp.jpg
```

## Główna lekcja

Kluczowym błędem była różnica między tym, co widziała walidacja, a tym, co aplikacja finalnie zapisała.
