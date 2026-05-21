# PortSwigger Lab 01 - Remote Code Execution przez upload web shella

## Co testowałem

Testowałem, czy funkcja uploadu avatara pozwala wgrać plik PHP i wykonać go po stronie serwera.

## Co znalazłem

Aplikacja zaakceptowała plik PHP i zapisała go w katalogu avatarów dostępnym z poziomu weba.

Przykład:

```text
/files/avatars/shell.php?cmd=cat+/home/carlos/secret
```

## Przyczyna źródłowa

Aplikacja pozwalała wgrywać wykonywalne pliki server-side i serwowała je z miejsca, w którym serwer webowy je wykonywał.

## Wpływ

Atakujący mógł wykonywać komendy na serwerze i czytać wrażliwe pliki.

## Remediacja

- odrzucaj wykonywalne rozszerzenia
- przechowuj uploady poza wykonywalnym web rootem
- wyłącz wykonywanie skryptów w katalogach uploadu
- generuj bezpieczne nazwy plików po stronie serwera
- waliduj typ pliku po stronie serwera

## Lekcja dla developera

Nigdy nie pozwalaj, żeby pliki wgrywane przez użytkownika były wykonywane jako kod.

## Główna lekcja

Niebezpieczny był nie tylko sam upload. Kluczowe było to, że uploadowany plik PHP mógł zostać wykonany.
