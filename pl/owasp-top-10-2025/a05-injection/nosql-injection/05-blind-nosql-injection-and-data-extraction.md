# Blind NoSQL Injection and Data Extraction

## Idea

Blind NoSQL Injection występuje wtedy, gdy aplikacja jest podatna na injection, ale nie zwraca sekretu bezpośrednio.

Zamiast tego odpowiedź ujawnia, czy zadany warunek jest prawdziwy albo fałszywy.

```text
warunek prawdziwy -> odpowiedź A
warunek fałszywy  -> odpowiedź B
```

Taka różnica tworzy oracle.

## Znalezienie oracle

Najpierw trzeba ustalić stabilny baseline:

- normalny request,
- request z inputem prawdziwym,
- request z inputem fałszywym,
- marker odpowiedzi, który jest powtarzalny.

Markerem może być:

- obecność rekordu,
- inna treść JSON,
- inny status,
- inna długość odpowiedzi,
- komunikat błędu,
- redirect albo jego brak.

## Ekstrakcja długości

Zanim testuje się znaki, warto ustalić długość sekretu.

Przykład koncepcyjny:

```text
Czy password length == 1?
Czy password length == 2?
Czy password length == 3?
```

Poprawna długość to ta, która daje odpowiedź true.

## Ekstrakcja znak po znaku

Następnie testuje się pozycję i znak.

```text
Czy password[0] == "a"?
Czy password[0] == "b"?
Czy password[1] == "a"?
```

Dla każdej pozycji jeden znak powinien dać marker true.

## Syntax Injection approach

Gdy injection point jest JavaScript-style expression, warunek może odnosić się do pola dokumentu, na przykład koncepcyjnie:

```text
this.password.length == 8
this.password[0] == "a"
```

Najważniejsze jest nie kopiowanie payloadu, ale rozumienie:

```text
input -> wyrażenie query -> warunek true/false -> marker odpowiedzi
```

## Operator Injection approach

Gdy aplikacja akceptuje operatory, `$regex` może działać jako oracle.

Przykład koncepcyjny:

```json
{
  "password": {
    "$regex": "^a"
  }
}
```

Jeżeli `^a` daje inny marker niż `^b`, można testować kolejne prefiksy.

## Automatyzacja

Automatyzacja ma sens dopiero po ręcznym potwierdzeniu:

- podatnego endpointu,
- poprawnego parametru,
- warunku true,
- warunku false,
- stabilnego markera odpowiedzi,
- poprawnego encodingu.

W Burp Intruder:

- Cluster Bomb testuje wszystkie kombinacje payloadów,
- Pitchfork paruje payloady według pozycji list.

Do testów pozycji i znaków zwykle potrzebne są wszystkie kombinacje.

## Remediacja

- Usuń możliwość wstrzykiwania operatorów lub składni query.
- Egzekwuj typy i schematy.
- Nie przechowuj haseł jako plaintext ani nie odpytuj ich w query.
- Weryfikuj hasła przez hash comparison poza query.
- Normalizuj odpowiedzi zależne od sekretów.
- Dodaj rate limiting i monitoring powtarzalnych prób.
