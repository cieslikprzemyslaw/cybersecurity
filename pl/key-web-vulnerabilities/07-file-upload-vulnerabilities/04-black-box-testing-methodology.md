# Metodologia testowania uploadu metodą black-box

## TL;DR

Nie zaczynaj od payloadu. Najpierw zrozum, co aplikacja akceptuje, blokuje, zapisuje i serwuje.

## Kroki

1. Rozpoznaj technologię: `Server`, `X-Powered-By`, framework, komunikaty błędów.
2. Znajdź punkty uploadu.
3. Sprawdź walidację po stronie klienta.
4. Wgraj bezpieczny poprawny plik.
5. Znajdź finalny URL i miejsce serwowania.
6. Porównaj pliki zaakceptowane i odrzucone.
7. Zidentyfikuj filtr: rozszerzenie, MIME, magic bytes, rozmiar pliku, nazwa pliku.
8. Sprawdź, czy plik jest treścią statyczną, czy jest wykonywany.
9. Oddziel akceptację uploadu od wykonania kodu.
10. Sprawdź zachowanie specyficzne dla stacku.
11. Myśl jak developer: przyczyna źródłowa, wpływ, remediacja, test regresji.

## Macierz testów

```text
test.jpg          -> zaakceptowany
shell.php         -> odrzucony
shell.php.jpg     -> zaakceptowany, ale niewykonany
shell.jpg.php     -> odrzucony
shell.php%00.jpg  -> zaakceptowany i zapisany inaczej
```

## Akceptacja vs wykonanie

Zaakceptowany upload nie oznacza automatycznie podatności możliwej do wykorzystania.

Przykład obserwacji:

```text
shell.php  -> odrzucony
shell.php7 -> zaakceptowany, ale serwowany jako statyczny tekst
shell.l33t -> zaakceptowany i wykonany po zmianie konfiguracji serwera
```

Dla każdego zaakceptowanego niebezpiecznie wyglądającego pliku sprawdź, jak serwer go obsługuje:

- renderuje jako obraz
- pobiera jako plik
- serwuje jako tekst źródłowy
- wykonuje jako kod po stronie serwera

## Zachowanie zależne od stacku

Fingerprint technologii pomaga wybrać następną gałąź testów.

Przykłady:

- Apache może sprawić, że `.htaccess` będzie istotny, jeśli lokalne override'y konfiguracji są włączone.
- Nginx nie używa `.htaccess`, więc ta sama ścieżka zwykle nie ma sensu.
- wykonanie PHP zależy od konfiguracji serwera webowego i handlera PHP.
- Cloud object storage zwykle serwuje pliki jako treść statyczną, ale może wprowadzić ryzyka publicznego dostępu i stored XSS.

Wzorzec bypassu blacklisty w Apache:

```text
1. Wgraj `.htaccess` z `AddType application/x-httpd-php .custom`
2. Wgraj payload PHP jako `shell.custom`
3. Request do `/files/avatars/shell.custom?cmd=whoami`
```

Kluczowa lekcja:

```text
Zaakceptowany upload nie oznacza wykonania kodu.
Odrzucony upload nie oznacza, że wszystkie ścieżki do wykonania kodu są zamknięte.
```

## Główna lekcja

Testowanie uploadu metodą black-box opiera się na enumeracji i porównywaniu zachowania aplikacji.
