# NoSQL Operator Injection

## Czym jest Operator Injection?

Operator Injection występuje wtedy, gdy input użytkownika jest interpretowany jako operator NoSQL albo zagnieżdżony obiekt query.

Typowy błąd:

```text
aplikacja oczekuje stringa
backend akceptuje obiekt
obiekt zawiera operator query
query zmienia znaczenie
```

## Przykład mentalny

Bezpieczne oczekiwanie:

```json
{
  "username": "alice",
  "password": "secret"
}
```

Niebezpieczny input:

```json
{
  "username": {
    "$ne": null
  },
  "password": {
    "$ne": null
  }
}
```

Jeżeli backend przekaże taki obiekt bez walidacji do filtra, query może znaczyć:

```text
znajdź użytkownika, którego username nie jest null
i password nie jest null
```

W źle zaprojektowanym logowaniu może to zwrócić pierwszy pasujący dokument.

## Typowe operatory

Przy sprawdzaniu nazw operatorów i ich zachowania używaj oficjalnej dokumentacji MongoDB [Query Predicates](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/).

- `$ne` - wartość nie jest równa,
- `$nin` - wartość nie należy do listy,
- `$gt` - wartość jest większa,
- `$regex` - wartość pasuje do wyrażenia regularnego.

## Auth bypass

Operator Injection w loginie jest szczególnie ryzykowne, gdy:

- backend buduje filtr bezpośrednio z request body,
- `username` i `password` nie są egzekwowane jako stringi,
- hasło jest częścią query zamiast porównania z hashem,
- sukces logowania zależy tylko od tego, czy query zwróciło dokument.

Bezpieczniejszy model:

```text
1. zwaliduj username/email jako string
2. pobierz użytkownika po dokładnym username/email
3. porównaj podane hasło z hashem poza query
4. odrzuć obiekty, tablice i nieznane pola
```

## `$regex` jako oracle

Jeżeli atakujący może użyć `$regex`, może testować warunki o sekrecie.

Przykład koncepcyjny:

```json
{
  "password": {
    "$regex": "^a"
  }
}
```

Jeżeli odpowiedź różni się dla dopasowania i braku dopasowania, endpoint może stać się oracle true/false.

## Dowody Operator Injection

- Obiekt w miejscu stringa zmienia odpowiedź.
- `$ne` powoduje inne zachowanie niż zwykły tekst `"$ne"`.
- Login albo lookup zwraca rekord bez poprawnych danych.
- `$regex` pozwala rozróżnić warunki prawdziwe i fałszywe.
- Backend akceptuje bracket notation typu `username[$ne]=x`.

## Remediacja

- Waliduj schemat requestu przed query construction.
- Egzekwuj typy prymitywne dla pól logowania i lookupu.
- Odrzucaj zagnieżdżone obiekty tam, gdzie nie są potrzebne.
- Nie przekazuj całego request body bezpośrednio do filtra bazy.
- Nie pozwalaj klientowi wybierać operatorów query.
- Nie weryfikuj haseł przez query z plaintext password.
