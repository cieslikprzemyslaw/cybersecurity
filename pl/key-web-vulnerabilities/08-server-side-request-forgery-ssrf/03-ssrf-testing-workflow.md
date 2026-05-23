# Workflow testowania SSRF

## Cel

Używaj tego workflow w legalnych labach i autoryzowanym testowaniu, gdy sprawdzasz potencjalne zachowanie SSRF.

Celem jest zrozumienie, czy input kontrolowany przez użytkownika może sprawić, że backend wyśle request do niezamierzonego celu.

## Podstawowy workflow

### 1. Znajdź parametr kandydujący

Szukaj parametrów, które zawierają albo wpływają na:

- pełne URL-e,
- hostnames,
- ścieżki,
- nazwy serwisów,
- callback URLs,
- webhook URLs,
- URL-e obrazów lub dokumentów.

Przykład:

```http
stockApi=http://stock.example.internal/product/stock/check?productId=1&storeId=1
```

### 2. Potwierdź normalne zachowanie

Wyślij request normalnie i zapisz:

- status code,
- response body,
- długość odpowiedzi,
- typ odpowiedzi,
- oczekiwaną funkcjonalność.

Przykład baseline:

```text
Normalny stock check zwraca liczbę, np. 350.
```

Baseline jest ważny, bo testowanie SSRF polega na porównywaniu różnic.

### 3. Zdekoduj input URL

Jeżeli parametr jest URL-encoded, zdekoduj go, żeby zrozumieć rzeczywisty cel backendowy.

Encoded:

```text
http%3A%2F%2Fstock.example.internal%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
```

Decoded:

```text
http://stock.example.internal:8080/product/stock/check?productId=1&storeId=1
```

### 4. Sprawdź, czy backend pobiera URL

Zmień URL w kontrolowany sposób i porównaj odpowiedź.

W legalnym labie mogą to być testy:

```text
http://localhost/
http://127.0.0.1/
http://127.1/
```

Nie używaj takich testów przeciwko realnym systemom bez autoryzacji.

### 5. Sprawdź, czy odpowiedź wraca do użytkownika

Jeżeli aplikacja zwraca odpowiedź backendu, jest to regular / non-blind SSRF.

Sygnały:

- HTML z wewnętrznej strony pojawia się w odpowiedzi,
- widać treść wewnętrznego panelu admina,
- status, długość albo tytuł strony wyraźnie się zmienia,
- pojawiają się wewnętrzne komunikaty błędów.

Jeżeli odpowiedź nie wraca, może to być blind SSRF.

### 6. Ustal, czy celem jest localhost czy inny backend

SSRF może celować w:

```text
localhost / 127.0.0.1
```

albo w inny system wewnętrzny:

```text
192.168.0.X:8080
10.X.X.X
172.16-31.X.X
```

Drugi przypadek jest ważny, bo pokazuje, że podatny backend może działać jak most do wewnętrznej sieci.

### 7. Zacznij od Burp Repeater

Najpierw użyj Burp Repeater, żeby zrozumieć:

- co się zmienia,
- co jest blokowane,
- co jest pobierane,
- które różnice w odpowiedzi mają znaczenie.

Repeater jest najlepszy do zrozumienia zachowania.

### 8. Użyj Intrudera, gdy jest zakres

Używaj Burp Intruder w legalnych labach, gdy trzeba przetestować wiele wartości, np.:

- zakres IP wewnętrznych,
- zakres portów,
- listę ścieżek,
- listę hostów.

Przykład wzorca zakresu wewnętrznego:

```text
http://192.168.0.§1§:8080/admin
```

Typ payloadu:

```text
Numbers: 1 to 255
```

Porównuj:

- status code,
- długość odpowiedzi,
- tytuł strony,
- body odpowiedzi,
- słowa kluczowe takie jak `Admin`, `Users`, `Delete`.

Praktyczna reguła:

> SSRF + zakres IP = Intruder.

### 9. Najpierw zmapuj filtr

Jeżeli aplikacja blokuje request, nie przechodź od razu do losowych payloadów.

Najpierw ustal, co jest blokowane:

- `localhost`?
- `127.0.0.1`?
- prywatne zakresy IP?
- `/admin`?
- konkretne protokoły?
- konkretne porty?
- redirecty?
- wartości zakodowane?

Przykład:

```text
localhost -> zablokowany
127.0.0.1 -> zablokowany
127.1 -> dozwolony
/admin -> zablokowany
/%2561dmin -> dozwolony
```

To pokazuje słabą blacklistę, a nie solidną walidację celu.

### 10. Potwierdź wpływ bezpieczeństwa

Dobre znalezisko SSRF powinno opisywać wpływ, a nie tylko payload.

Pytania o wpływ:

- Czy backend mógł osiągnąć `localhost`?
- Czy mógł uzyskać dostęp do wewnętrznego panelu admina?
- Czy mógł dotrzeć do innego backendu?
- Czy mógł wywołać akcję zmieniającą stan?
- Czy mógł enumerować wewnętrzne hosty albo porty?
- Czy mógł osiągnąć cloud metadata?
- Czy odpowiedź jest widoczna, czy blind?

### 11. Myśl jak developer

Po potwierdzeniu problemu zapytaj:

- Dlaczego input użytkownika mógł definiować request backendowy?
- Dlaczego backend ufał `localhost` albo usługom wewnętrznym?
- Dlaczego wewnętrzny panel admina nie miał właściwego uwierzytelnienia?
- Czy ochrona była oparta na stringowej blacklistcie?
- Czy walidowano finalny rozwiązany cel?
- Czy redirecty były obsługiwane bezpiecznie?

## Porównywanie odpowiedzi

Podczas porównywania odpowiedzi SSRF sprawdzaj:

```text
Status code
Response length
Content-Type
Page title
HTML content
Error message
Redirect location
Response time
```

Jedna różniąca się odpowiedź może zidentyfikować wewnętrzny cel.

## Typowy błąd

Bezpośredni request do wewnętrznego endpointu admina może się nie udać:

```http
GET /admin/delete?username=carlos
```

Może zwrócić `401 Unauthorized`, bo request pochodzi od użytkownika lub z przeglądarki.

Atak SSRF musi przejść przez podatny parametr requestu kontrolowanego przez backend:

```http
POST /product/stock

stockApi=http://localhost/admin/delete?username=carlos
```

Różnicą jest źródło requestu.

## Przypomnienie o zakresie

Testuj SSRF tylko w:

- legalnych labach,
- własnych środowiskach lokalnych,
- wyraźnie autoryzowanych środowiskach testowych,
- scope'ach bug bounty, w których testowanie SSRF jest dozwolone.

Skanowanie wewnętrznych IP, portów i endpointów metadata może być wysokiego ryzyka poza autoryzowanymi labami.

## Główna lekcja

Testowanie SSRF nie polega na zapamiętaniu jednego payloadu.

Chodzi o zrozumienie:

```text
cel kontrolowany przez użytkownika -> request backendu -> przekroczona granica zaufania -> wewnętrzny wpływ
```
