# A05: Injection - Learning Notes

## Szerszy model mentalny

SQL Injection i NoSQL Injection pomogły mi zrozumieć szerszy model A05:

```text
kontrolowany input -> interpreter/query engine -> zmienione zachowanie
```

Payload nie jest sam w sobie przyczyną źródłową. Podatność istnieje, gdy aplikacja pozwala, aby niezaufany input stał się składnią query, operatorem query albo częścią wykonywalnego wyrażenia.

## Lekcje SQL Injection

- Pojedynczy apostrof może zepsuć zapytanie SQL, gdy input trafia do kontekstu stringa.
- Komentarz SQL może naprawić zapytanie przez zakomentowanie pozostałego SQL.
- `UNION SELECT` wymaga tej samej liczby kolumn co oryginalne zapytanie.
- `NULL` jest przydatny przy testowaniu liczby kolumn.
- Blind SQLi wymaga wiarygodnej wyroczni true/false, a nie bezpośrednio wyświetlonych danych.

## Lekcje NoSQL Injection

### NoSQL to nie jeden język zapytań

Różne bazy NoSQL używają różnych modeli danych i systemów query. W MongoDB zapytania często używają obiektów strukturalnych albo tablic asocjacyjnych.

### Wartość może stać się logiką query

Najważniejsze pytanie:

```text
Czy mój input nadal jest prostą wartością,
czy stał się zagnieżdżonym obiektem albo operatorem query?
```

Pole logowania, które powinno zawierać string, może być niebezpieczne, jeśli backend akceptuje strukturę typu:

```php
['$ne' => 'test']
```

### Operator Injection i Syntax Injection to różne problemy

Operator Injection zmienia query przez strukturalne operatory, takie jak `$ne`, `$nin` albo `$regex`.

Syntax Injection wychodzi z wyrażenia query i dodaje nową składnię. W MongoDB jest to zwykle rzadsze i może dotyczyć custom JavaScript-style query logic, na przykład `$where`.

### Background endpoints mają znaczenie

W labie z ekstrakcją danych widoczna strona była:

```text
/my-account?id=wiener
```

Podatny request danych był requestem w tle:

```text
/user/lookup?user=wiener
```

To utrwaliło, że w AppSec trzeba sprawdzać HTTP history, fetch/XHR traffic i API calls, a nie tylko pasek adresu.

### Blind extraction to problem oracle

Endpoint nie zwracał hasła bezpośrednio. Ujawniał, czy zdanie o haśle jest prawdziwe.

Przykładowe pytania koncepcyjne:

```text
Czy hasło administratora ma długość 8?
Czy znak na indeksie 0 to a?
Czy znak na indeksie 1 to b?
```

Jedna odpowiedź oznaczała true, inna false.

## Lekcja o encodingu

Raw payloady i encoded payloady nie są wymienne.

- Raw payload może być właściwy w UI, które samo wykonuje encoding.
- Encoded characters są potrzebne przy bezpośredniej edycji request target.
- `&` jest szczególnie ważne, bo niezakodowany ampersand może rozdzielić parametry.
- Podwójny albo błędny encoding może zamienić zamierzoną składnię w literalny tekst.

## Błędy i korekty

- Najpierw testowałem endpoint `/login`, bo cel dotyczył hasła administratora. Podatna funkcja w labie nie była requestem logowania.
- Potem testowałem `/my-account?id=...`, ale ta trasa zwracała redirecty i nie była faktycznym user lookup.
- Prawdziwy request `/user/lookup` znalazłem w Burp HTTP history.
- Początkowo próbowałem umieścić regex string w wyrażeniu bez zastosowania go do `this.password`.
- Próbowałem tworzyć osobny parametr query `password`, ale podatny endpoint używał tylko wyrażenia w `user`.
- Miałem request z błędnym i powtarzanym encodingiem, na przykład sekwencjami `%u...`.
- Nauczyłem się, że `this.password[0]` indeksuje pierwszy znak stringa JavaScript; hasło nie musi być tablicą.
- Nauczyłem się, dlaczego Cluster Bomb pasował do każdej kombinacji pozycji hasła i litery, a Pitchfork parowałby payloady zamiast testować pełny zestaw kombinacji.

## Aktualne rozumienie

Powinienem umieć wyjaśnić:

- jaki input kontroluję,
- który endpoint naprawdę go przetwarza,
- czy input jest wartością, obiektem, operatorem albo składnią,
- jaki parser albo query engine go przetwarza,
- jaki dowód pokazuje, że query się zmieniło,
- jak oracle może ujawniać dane bez bezpośredniego wyświetlania,
- jaka jest przyczyna źródłowa,
- jak developer powinien to naprawić,
- jakie testy regresji powinny zapobiec powrotowi problemu.

Nie muszę pamiętać każdego operatora albo payloadu. Muszę rozumieć struktury danych, budowę query, dowody, impact i remediację.
