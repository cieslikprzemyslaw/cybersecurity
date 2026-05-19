# Core Concepts - Path Traversal / File Access Bugs

## Czym jest Path Traversal?

Path Traversal to podatność, w której input użytkownika wpływa na ścieżkę pliku po stronie serwera, a backend odczytuje plik spoza dozwolonego katalogu.

Ważne: atakujący nie przegląda serwera tak jak w terminalu komendą `cd`. Atakujący manipuluje ścieżką w requestcie, a aplikacja odczytuje plik za niego.

## File name vs file path

File name to sama nazwa pliku:

```txt
avatar.jpg
invoice.pdf
config.php
```

File path opisuje, gdzie plik znajduje się w systemie:

```txt
/var/www/images/avatar.jpg
../../config.php
/etc/passwd
```

Bezpieczniejszy design powinien unikać sytuacji, w której użytkownik kontroluje surową ścieżkę do pliku.

## Katalog bazowy

Base directory to katalog, z którego aplikacja spodziewa się czytać pliki.

Przykład:

```txt
/var/www/images/
```

Aplikacja może chcieć czytać tylko obrazki z tego katalogu.

Podatny wzorzec:

```txt
/var/www/images/ + userInput
```

Jeżeli `userInput` to `../../../etc/passwd`, finalna ścieżka może wyjść poza katalog images.

## Relative path

Relative path zależy od aktualnego katalogu albo base directory.

Przykłady:

```txt
avatar.jpg
../config.php
../../etc/passwd
```

## Absolute path

Absolute path zaczyna się od root filesystemu.

Linux:

```txt
/etc/passwd
/var/www/images/avatar.jpg
```

Windows:

```txt
C:\Windows\win.ini
C:\Users\Public\file.txt
```

Czasami aplikacja blokuje `../`, ale nadal akceptuje absolute path, np. `/etc/passwd`.

## Co oznacza `../`?

`../` oznacza "idź jeden katalog wyżej".

Przykład:

```txt
/var/www/images/../
```

rozwiązuje się do:

```txt
/var/www/
```

Kilka sekwencji `../` może wyjść kilka poziomów wyżej.

## Canonical / resolved path

Canonical albo resolved path to finalna, rzeczywista ścieżka po normalizacji elementów takich jak `../`, `./` i podwójne slashe.

Przykład:

```txt
/var/www/images/../../../etc/passwd
```

może rozwiązać się do:

```txt
/etc/passwd
```

To ważne, bo security check powinien sprawdzać finalną ścieżkę, a nie tylko raw input.

## Dlaczego słabe string checks zawodzą

Sprawdzenie:

```txt
Czy input zaczyna się od /var/www/images/?
```

nie wystarczy, jeśli aplikacja nie sprawdza finalnej ścieżki.

Przykład:

```txt
/var/www/images/../../../etc/passwd
```

Input zaczyna się od `/var/www/images/`, ale finalna ścieżka może wskazywać na `/etc/passwd`.

## Mental model

```txt
request -> parametr -> logika plików w backendzie -> filesystem -> response
```

Path Traversal występuje wtedy, gdy input użytkownika trafia do logiki dostępu do plików bez poprawnej walidacji finalnej resolved path.
