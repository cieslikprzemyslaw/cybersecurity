# PortSwigger 04 - Superfluous URL Decode / Double Encoding

## Co testowałem

Testowałem endpoint, w którym normalny traversal i single URL encoding nie działały.

## Co znalazłem

Zadziałał double URL encoding:

```http
GET /image?filename=..%252f..%252f..%252fetc%252fpasswd
```

## Dlaczego to ma znaczenie

`%252f` dekoduje się najpierw do `%2f`, a potem do `/`.

```txt
%252f -> %2f -> /
```

Po dwóch krokach decode payload staje się:

```txt
../../../etc/passwd
```

## Root cause

Aplikacja prawdopodobnie walidowała albo filtrowała input przed późniejszym decode, który stworzył poprawną traversal sequence.

## Wpływ

Atakujący mógł ominąć wcześniejszą walidację i uzyskać dostęp do wrażliwych plików.

## Notatka dla mnie

Błędy encodingu często biorą się z tego, że walidacja dzieje się przed finalnym decode step aplikacji.
