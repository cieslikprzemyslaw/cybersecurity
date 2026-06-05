# MongoDB Data Model and Query Filters

## Po co ten plik

NoSQL Injection w MongoDB jest łatwiejsze do zrozumienia, gdy widać różnicę między zwykłą wartością a strukturą query.

W SQLi często patrzę na tekst zapytania. W MongoDB muszę też patrzeć na typ danych i kształt obiektu, który trafia do drivera.

## Dokumenty i kolekcje

MongoDB zapisuje rekordy jako dokumenty BSON/JSON-like.

```json
{
  "username": "carlos",
  "role": "user",
  "enabled": true
}
```

Dokumenty są przechowywane w kolekcjach. Kolekcja `users` może zawierać wiele dokumentów użytkowników.

## Query filter

Typowy filtr query wygląda jak obiekt:

```json
{
  "username": "carlos"
}
```

Znaczenie:

```text
znajdź dokumenty, gdzie username równa się "carlos"
```

Jeżeli backend buduje taki filtr tylko z pól i operatorów kontrolowanych po stronie serwera, input użytkownika pozostaje wartością.

## Operatory query

MongoDB używa operatorów zaczynających się od `$`, na przykład:

- `$ne` - not equal,
- `$nin` - not in,
- `$gt` - greater than,
- `$regex` - dopasowanie regex,
- `$where` - custom JavaScript expression.

Oficjalna dokumentacja MongoDB [Query Predicates](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/) jest punktem odniesienia dla nazw operatorów, ich zachowania i różnic zależnych od wersji albo kontekstu query.

Przykład:

```json
{
  "username": {
    "$ne": "carlos"
  }
}
```

Znaczenie:

```text
znajdź dokumenty, gdzie username nie jest równe "carlos"
```

## Niebezpieczna zmiana typu

Bezpieczne oczekiwanie:

```json
{
  "username": "carlos"
}
```

Niebezpieczny rezultat, jeżeli backend zaakceptuje obiekt w miejscu stringa:

```json
{
  "username": {
    "$ne": null
  }
}
```

Problem nie polega tylko na znakach specjalnych. Problemem jest to, że input zmienił typ i stał się strukturą query.

## URL-encoded form notation

Niektóre parsery formularzy potrafią zamienić zapis:

```text
username[$ne]=test
```

na strukturę podobną do:

```json
{
  "username": {
    "$ne": "test"
  }
}
```

Dlatego testując NoSQL Injection, trzeba sprawdzać nie tylko JSON body, ale też query string i URL-encoded form bodies.

## Bezpieczny model

- Backend wybiera pola query.
- Backend wybiera dozwolone operatory.
- Input użytkownika jest walidowany jako konkretny typ.
- Nieznane pola są odrzucane albo usuwane.
- Obiekty i tablice są odrzucane tam, gdzie oczekiwany jest string.

## Najważniejsza lekcja

W MongoDB attack surface często wynika z parsera i struktury danych. Pytanie brzmi:

```text
Czy input jest wartością,
czy stał się częścią obiektu query?
```
