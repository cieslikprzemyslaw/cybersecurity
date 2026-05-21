# Ściąga z walidacji i bypassów filtrów

## TL;DR

Walidacja uploadu często składa się z kilku słabych kontroli. Jedna kontrola rzadko wystarcza.

## Walidacja po stronie klienta

Walidacja po stronie klienta działa w przeglądarce. Pomaga UX, ale nie jest granicą bezpieczeństwa.

### Cztery bypassy walidacji po stronie klienta

1. Wyłącz JavaScript.
2. Zmodyfikuj odpowiedź z kodem strony w Burp.
3. Zmodyfikuj request uploadu w Burp.
4. Wyślij request bezpośrednio do endpointu uploadu.

## Walidacja po stronie serwera

Walidację po stronie serwera testujemy przez enumerację.

## Przypadki testowe walidacji rozszerzeń

```text
shell.php
shell.phtml
shell.phar
shell.php5
shell.jpg.php
shell.php.jpg
shell.PHP
```

## Blacklista vs allowlista

Blacklista blokuje znane złe rozszerzenia. Jest krucha, bo trzeba przewidzieć każdy niebezpieczny wariant.

Allowlista pozwala tylko na konkretne oczekiwane rozszerzenia, np. `.jpg`, `.jpeg`, `.png`, `.webp`.

## Walidacja MIME / Content-Type

Słaby serwer może ufać `Content-Type` części pliku:

```http
Content-Type: image/png
```

To można zmienić w Burp.

## Magic bytes

```text
PNG  -> 89 50 4E 47 0D 0A 1A 0A
JPEG -> FF D8 FF DB
```

Magic bytes są lepsze niż samo rozszerzenie albo `Content-Type`, ale nie są pełną ochroną.

## Bypass przez zaciemnione rozszerzenie pliku

Przypadki testowe do legalnych labów:

```text
shell.php
shell.php.jpg
shell.jpg.php
shell.php%00.jpg
shell.php%00.png
shell.PHP
shell.pHp
shell.php%2ejpg
shell%2ephp.jpg
```

Cel nie polega na zapamiętaniu payloadów. Chodzi o zrozumienie, jak nazwa pliku jest obsługiwana na różnych etapach:

1. walidacja
2. zapis
3. finalny URL
4. wykonanie po stronie serwera

### Obserwacja z PortSwigger

```text
shell.php         -> odrzucony
shell.php.jpg     -> zaakceptowany, ale serwowany jako obraz
shell.jpg.php     -> odrzucony
shell.php%00.jpg  -> zaakceptowany i zapisany jako shell.php
```

## Główna lekcja

Walidacja uploadu musi być warstwowa. Najważniejsze są bezpieczny zapis i wyłączenie wykonywania kodu w katalogu uploadu.
