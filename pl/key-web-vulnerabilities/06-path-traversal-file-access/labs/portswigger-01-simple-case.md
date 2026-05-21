# PortSwigger 01 - File Path Traversal, Simple Case

## Co testowałem

Testowałem endpoint ładujący obrazek:

```http
GET /image?filename=4.jpg
```

## Co znalazłem

Zmiana filename na losową wartość zwróciła:

```txt
"No such file"
```

Basic traversal zwrócił `/etc/passwd`:

```http
GET /image?filename=../../../etc/passwd
```

## Dlaczego to ma znaczenie

Backend używał parametru `filename` kontrolowanego przez użytkownika do odczytu pliku z filesystemu serwera.

## Przyczyna źródłowa

Aplikacja nie walidowała finalnej resolved path przed odczytem pliku.

## Wpływ

Atakujący mógł odczytać lokalne, wrażliwe pliki z serwera.

## Notatka dla mnie

`filename` wyglądało niewinnie, bo ładowało obrazek, ale nadal trafiało do logiki dostępu do plików.
