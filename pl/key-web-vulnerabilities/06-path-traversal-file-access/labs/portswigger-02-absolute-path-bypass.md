# PortSwigger 02 - Absolute Path Bypass

## Co testowałem

Testowałem endpoint ładujący obrazek:

```http
GET /image?filename=13.jpg
```

Losowa wartość zwróciła:

```txt
"No such file"
```

## Co znalazłem

Zamiast `../`, zadziałała absolute path:

```http
GET /image?filename=/etc/passwd
```

## Dlaczego to ma znaczenie

Blokowanie traversal sequences takich jak `../` nie wystarcza, jeśli backend nadal akceptuje pełne ścieżki filesystemu.

## Przyczyna źródłowa

Aplikacja ufała parametrowi `filename` i nie sprawdzała finalnej resolved path.

## Wpływ

Atakujący mógł odczytać pliki, podając absolute paths.

## Notatka dla mnie

Jeśli `../` jest blokowane, warto sprawdzić, czy backend przyjmie bezpośrednio absolute path.
