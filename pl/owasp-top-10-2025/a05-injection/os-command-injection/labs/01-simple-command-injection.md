# Lab 1 - Simple OS Command Injection

## Platforma

PortSwigger Web Security Academy

Lab: https://portswigger.net/web-security/os-command-injection/lab-simple

## Zakres

Wyłącznie autoryzowany lab treningowy.

## Cel labu

Użyć funkcji sprawdzania stanu magazynowego do wykonania nieszkodliwej komendy i identyfikacji użytkownika systemowego, jako którego działa proces aplikacji.

## Podatna funkcja

```text
POST /product/stock
```

Request zawierał:

```text
productId=1&storeId=1
```

## Kontrolowany input

Obie wartości były kontrolowane przez klienta, ale do testu wybrano `storeId`.

## Początkowa hipoteza

Stock checker mógł wywoływać legacy script albo komendę, używając identyfikatorów produktu i sklepu jako argumentów.

Możliwa komenda koncepcyjna:

```text
stock-report <productId> <storeId>
```

Dokładna komenda backendowa nie była widoczna, więc pozostało to hipotezą.

## Kontrolowany test

Wartość `storeId` została zmieniona tak, aby separator shella wprowadził nieszkodliwą komendę `whoami`.

Odpowiedź zawierała:

```text
62
peter-H8uEby
```

## Dowód

- `62` było normalną wartością stock.
- `peter-H8uEby` było rozpoznawalnym outputem z `whoami`.
- Obie wartości pojawiły się w tej samej odpowiedzi HTTP.

To był direct albo verbose OS Command Injection.

## Co wynik potwierdził

```text
kontrolowany storeId
-> separator shella został zinterpretowany
-> dodatkowa komenda wykonała się
-> stdout wrócił w odpowiedzi HTTP
```

Udany separator mocno sugerował interpretację shell-like, ale nie identyfikował dokładnej implementacji shella.

## Przyczyna źródłowa

Input kontrolowany przez użytkownika został włączony do komendy interpretowanej przez shell bez zachowania separacji między danymi a składnią komendy.

## Impact

Arbitrary command execution jako użytkownik procesu aplikacji może ujawnić dane czytelne dla aplikacji i stworzyć ścieżkę do szerszego kompromisu, zależnie od uprawnień i dostępu środowiska.

## Remediacja

- Zastąp zewnętrzną komendę bezpiecznym internal API tam, gdzie to możliwe.
- Jeśli proces jest wymagany, użyj stałego executable i przekaż product/store IDs jako osobne zwalidowane argumenty.
- Nie wywołuj shella.
- Egzekwuj typ liczbowy i zakres.
- Uruchamiaj proces z least privilege i limitami wykonania.
- Dodaj testy regresji dla separatorów, option-like values, timingu, plików i outbound interactions.

## Notatka z nauki

Ten lab był celowo prosty. Główna lekcja: direct command output jest wysokiej jakości dowodem, ale nadal trzeba powiązać go z jasnym data-flow explanation, a nie traktować tylko jako „działający payload”.
