# NoSQL Syntax Injection

## Czym jest Syntax Injection?

NoSQL Syntax Injection występuje wtedy, gdy input użytkownika zostaje wstawiony do wyrażenia query i może wyjść z zamierzonego kontekstu, a następnie dodać nową składnię.

W MongoDB ryzykownym miejscem jest custom JavaScript-style logic, na przykład `$where`.

MongoDB wymienia `$where` w oficjalnej referencji [Query Predicates](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/). W notatkach AppSec traktuj tę funkcję jako miejsce wymagające szczególnego review, bo pozwala na ocenę JavaScript-style expressions.

## Różnica względem Operator Injection

Operator Injection:

```text
input staje się obiektem albo operatorem query
```

Syntax Injection:

```text
input przerywa istniejące wyrażenie i dodaje własną składnię
```

## Test apostrofem

Jeżeli input trafia do stringa w wyrażeniu, pojedynczy apostrof może przerwać składnię.

Sygnały:

- kontrolowany błąd,
- HTTP 500,
- komunikat MongoDB albo JavaScript,
- zmiana odpowiedzi względem baseline'u.

Sam błąd nie wystarcza do pełnego impactu, ale jest dobrym tropem.

## Warunki true/false

Po potwierdzeniu syntax-sensitive input trzeba porównać warunek prawdziwy i fałszywy.

Przykład koncepcyjny:

```text
input + warunek zawsze prawdziwy
input + warunek zawsze fałszywy
```

Jeżeli odpowiedzi są stabilnie różne, endpoint może pozwalać na manipulację query.

## Always-true condition

Warunek zawsze prawdziwy może rozszerzyć wynik poza zamierzony filtr, na przykład zwrócić:

- ukryte produkty,
- rekord innego użytkownika,
- konto administratora,
- dane spoza oczekiwanej kategorii.

## Blind Syntax Injection

Jeżeli endpoint nie zwraca danych bezpośrednio, ale pokazuje różnicę odpowiedzi, można zbudować boolean oracle.

Przykładowe pytania:

```text
Czy istnieje użytkownik administrator?
Czy hasło ma długość 8?
Czy pierwszy znak hasła to a?
```

To nadal jest injection, nawet jeśli dane nie są wyświetlane wprost.

## Dlaczego Syntax Injection jest mniej powszechne

W typowym MongoDB API developer może budować query jako obiekt, bez custom JavaScript expressions. Syntax Injection zwykle wymaga niebezpiecznego wzorca:

```text
string concatenation -> custom expression -> query engine
```

Mimo to jest ważne, bo potrafi prowadzić do blind extraction i kompromitacji kont.

## Remediacja

- Nie doklejaj inputu użytkownika do `$where`.
- Unikaj budowania custom JavaScript query.
- Używaj structured filters z polami i operatorami wybranymi przez backend.
- Waliduj typ i długość inputu.
- Zwracaj kontrolowane błędy bez stack trace i szczegółów interpretera.
- Normalizuj odpowiedzi, które mogłyby tworzyć oracle zależny od sekretu.
