# SSRF - gdzie testować

## Cel

Używaj tej checklisty podczas szukania miejsc podatnych na SSRF w legalnych labach, lokalnych aplikacjach treningowych albo autoryzowanym testowaniu.

Celem jest znalezienie miejsc, w których input kontrolowany przez użytkownika może wpływać na request wykonywany przez backend.

## Funkcje wysokiego ryzyka

SSRF najczęściej pojawia się w funkcjach, które pobierają, importują, podglądają, walidują albo przetwarzają zdalne zasoby.

Szukaj funkcji takich jak:

- sprawdzanie stanu magazynowego,
- pobieranie obrazów,
- import avatara z URL-a,
- generatory PDF,
- generatory screenshotów,
- podgląd linku / URL preview,
- konfiguracja webhooka,
- konfiguracja callback URL,
- import pliku z URL-a,
- konwertery dokumentów,
- import feedów,
- integracje z zewnętrznymi API,
- server-side analytics przetwarzające nagłówki `Referer`,
- parsery XML/dokumentów, które mogą pobierać zewnętrzne zasoby.

## Parametry warte sprawdzenia

Szukaj parametrów zawierających pełny URL:

```text
url=
uri=
link=
target=
endpoint=
callback=
webhook=
stockApi=
imageUrl=
avatarUrl=
feedUrl=
next=
redirect=
returnUrl=
```

Szukaj też parametrów zawierających tylko część URL-a:

```text
server=
host=
domain=
path=
api=
service=
resource=
```

SSRF nie zawsze wygląda jak pełny URL w query stringu. Czasem aplikacja przyjmuje tylko host, ścieżkę, nazwę serwisu albo ukryte pole formularza i dopiero po stronie serwera buduje finalny URL backendowy.

## Miejsca w request body

Sprawdzaj kandydatów na SSRF w:

- query string parameters,
- body formularza `POST`,
- JSON body,
- XML body,
- polach formularza multipart,
- ukrytych polach formularza,
- nagłówkach,
- cookies,
- zapisanych wartościach profilu lub ustawień.

Przykład body formularza:

```http
POST /product/stock
Content-Type: application/x-www-form-urlencoded

stockApi=http://stock.example.internal/product/stock/check?productId=1&storeId=1
```

Przykład JSON:

```json
{
  "imageUrl": "https://example.com/avatar.png"
}
```

Przykład ukrytego pola:

```html
<input type="hidden" name="avatar" value="/images/avatars/default.png">
```

## Ukryta powierzchnia ataku

Niektóre inputy SSRF nie są widoczne w pasku URL przeglądarki.

Sprawdź:

- DevTools Network tab,
- historię Burp Proxy,
- ukryte pola formularzy,
- requesty generowane przez JavaScript,
- wywołania API,
- JSON bodies,
- dokumenty XML,
- nagłówki takie jak `Referer`.

## Pytania podczas testowania

Podczas przeglądania requestu zapytaj:

1. Czy parametr zawiera URL, host, ścieżkę albo nazwę serwisu?
2. Czy backend pobiera dane z tej wartości?
3. Czy odpowiedź z tego requestu backendowego wraca do użytkownika?
4. Czy mogę zmienić host?
5. Czy mogę zmienić ścieżkę?
6. Czy w legalnym labie mogę osiągnąć `localhost` albo prywatny adres IP?
7. Czy jest filtr?
8. Czy filtr bazuje na blacklistcie, czy na ścisłej allowliście?
9. Czy backend podąża za redirectami?
10. Czy finalny rozwiązany cel pozostaje w dozwolonym zakresie?

## Typowe wzorce SSRF

### Parametr z pełnym URL-em

```http
stockApi=http://stock.example.internal/product/stock/check
```

Ryzyko:

```text
Atakujący może podmienić pełny cel backendowy.
```

### Parametr z hostem

```http
server=api
```

Backend buduje:

```text
https://api.example.internal/product/stock/check
```

Ryzyko:

```text
Atakujący może kontrolować hostname albo subdomenę używaną przez backend.
```

### Parametr tylko ze ścieżką

```http
path=/product/stock/check
```

Backend buduje:

```text
https://internal-api.example.local/product/stock/check
```

Ryzyko:

```text
Atakujący może użyć manipulacji ścieżką albo redirectów, jeśli walidacja jest słaba.
```

### Ukryte pole formularza

```html
<input type="hidden" name="avatar" value="/images/default.png">
```

Ryzyko:

```text
Ukryte pola nadal są kontrolowane przez użytkownika. Można je zmienić w DevTools albo Burp.
```

## Perspektywa frontendowa

Z perspektywy frontendu SSRF często kryje się za normalnymi funkcjami UI:

- "Check stock",
- "Import image",
- "Generate preview",
- "Fetch URL",
- "Connect webhook",
- "Validate endpoint".

UI może wyglądać niewinnie, ale pytanie bezpieczeństwa jest po stronie serwera:

> Co backend robi z tą wartością?

## Główna lekcja

Testowanie SSRF zaczyna się od znalezienia wartości kontrolowanych przez użytkownika, które wpływają na requesty backendowe.

Nie szukaj tylko oczywistych pełnych URL-i. Sprawdzaj też hosty, ścieżki, ukryte pola, wartości JSON, nagłówki i identyfikatory serwisów.
