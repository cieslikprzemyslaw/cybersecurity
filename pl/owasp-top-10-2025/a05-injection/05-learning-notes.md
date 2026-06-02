# A05: Injection - Learning Notes

## Czego nauczyłem się do tej pory

SQL Injection pomogło mi zrozumieć szerszy model A05 Injection:

```text
input -> interpreter -> zmienione zachowanie
```

Najważniejsza lekcja: problemem nie jest sam payload. Problem pojawia się wtedy, gdy aplikacja pozwala, aby dane użytkownika stały się częścią składni SQL.

## Ważne obserwacje SQL Injection

- Pojedynczy apostrof może zepsuć zapytanie SQL, gdy input trafia do kontekstu stringa.
- Komentarz SQL może naprawić zapytanie przez zakomentowanie pozostałej części oryginalnego SQL.
- `UNION SELECT` wymaga tej samej liczby kolumn co oryginalne zapytanie.
- `NULL` jest przydatny do testowania liczby kolumn, bo pasuje do wielu typów.
- `NULL,NULL` potwierdza strukturę, ale samo w sobie nie dowodzi ekstrakcji danych.
- Test z `'abc','def'` potwierdził, że kolumny tekstowe były widoczne w odpowiedzi HTML.
- Blind SQLi wymagało innego sposobu myślenia, bo dane nie były zwracane bezpośrednio.

## Lekcja o encodingu

W jednym zadaniu THM zakodowany payload pojawił się w wygenerowanym zapytaniu SQL jako literalny tekst, na przykład `%20`, `%27` albo `%7C`. To oznaczało, że payload nie został zdekodowany na oczekiwanej warstwie i był potraktowany jako tekst, a nie składnia SQL.

Ważna lekcja:

```text
Raw payload w UI, jeśli frontend koduje input.
Zakodowany payload w bezpośrednim URL albo Burp Repeater.
```

## Lekcja blind SQLi

Blind SQL Injection polega na zbudowaniu wiarygodnej wyroczni true/false.

W labie PortSwigger:

```text
Welcome back = TRUE
Brak Welcome back = FALSE
```

To pozwoliło potwierdzić tabelę users, potwierdzić użytkownika administrator, ustalić długość hasła i wyciągnąć hasło znak po znaku.

## Błędy i korekty

- Na początku zapomniałem, że komentarze SQL są ważne po zepsuciu zapytania apostrofem.
- Początkowo pomyliłem precyzyjną terminologię interpretera. W SQL Injection właściwym interpreterem jest baza danych / silnik SQL, a nie obiekt połączenia PHP.

## Aktualne rozumienie

Na moim obecnym poziomie AppSec powinienem umieć wyjaśnić:

- jaki input kontroluję,
- gdzie jest przetwarzany,
- jakie dowody potwierdzają SQLi,
- jaki jest realny wpływ,
- jak developer powinien to naprawić.

Nie muszę pamiętać każdego zaawansowanego payloadu SQLi dla każdej bazy, ale powinienem rozumieć mechanizm, impact i remediację.

Szczegółowe porównanie labów SQLi trzymam w [sql-injection/learning-summary.md](sql-injection/learning-summary.md), żeby ten plik pozostał skupiony na szerszym modelu A05.
