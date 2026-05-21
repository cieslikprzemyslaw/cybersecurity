# PortSwigger 03 - Traversal Sequences Stripped Non-Recursively

## Co testowałem

Testowałem endpoint ładujący obrazek:

```http
GET /image?filename=5.jpg
```

Basic payloads takie jak `/etc/passwd` i `../../../../etc/passwd` nie działały.

## Co znalazłem

Zadziałał nested traversal:

```http
GET /image?filename=....//....//....//etc/passwd
```

## Dlaczego to ma znaczenie

Aplikacja próbowała usuwać traversal sequences, ale filtr był słaby i nierekurencyjny.

## Przyczyna źródłowa

Backend prawdopodobnie usuwał `../` raz, ale nie sprawdzał ponownie wyniku po transformacji.

## Wpływ

Atakujący mógł ominąć słaby filtr i odczytać wrażliwe pliki.

## Notatka dla mnie

Gdy filtr usuwa wzorzec tylko raz, nested payload może odtworzyć groźny wzorzec już po filtrowaniu.
