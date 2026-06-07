# Detection and Fingerprinting

## Manual-first workflow

### 1. Zidentyfikuj funkcje

Szukaj funkcji, ktore generuja server-side output z wartosci kontrolowanych przez uzytkownika, na przyklad:

- komunikaty,
- email previews,
- CMS templates,
- user-customisable themes,
- generowanie faktur albo dokumentow,
- product/profile pages,
- error pages,
- zapisane szablony.

### 2. Zidentyfikuj kontrolowany input

Mozliwe zrodla:

- query parameters,
- pola formularza,
- JSON properties,
- headers,
- cookies,
- zapisane pola CMS,
- profile values,
- imported template content.

Nie zakladaj, ze najbardziej oczywisty parametr ID jest injection point.

### 3. Najpierw potwierdz reflection

Uzyj unikalnego markera:

```text
SSTI_TEST_123
```

Zapisz:

- czy sie pojawia,
- gdzie sie pojawia,
- czy jest encoded,
- czy zostal zmieniony,
- czy pojawia sie od razu czy pozniej.

Reflection potwierdza kontrole inputu, nie ewaluacje szablonu.

### 4. Przetestuj jedna hipoteze silnika

Uzyj jednego prostego, niedestrukcyjnego wyrazenia odpowiedniego dla podejrzanego engine.

Przed testem ustal:

- jakie zachowanie testujesz,
- dlaczego podejrzewasz ten engine,
- oczekiwany wynik przy ewaluacji,
- oczekiwany wynik przy literalnym wyswietleniu,
- co oznaczalby blad.

### 5. Porownaj wynik

| Wynik | Znaczenie |
|---|---|
| Literal expression | Brak ewaluacji, zla skladnia, zly kontekst albo escaping przed kompilacja |
| Calculated result | Mocny dowod server-side template evaluation |
| Engine-specific error | Przydatna wskazowka fingerprintingu |
| Generic server error | Przetwarzanie sie wywalilo, ale sam blad nie identyfikuje silnika |
| Timing albo side effect | Moze wspierac blind evaluation, ale wymaga powtarzalnych dowodow |

## Zrodla fingerprintingu

### Roznice skladni

Engines uzywaja roznych delimiterow i zasad wyrazen. Podobna skladnia nie gwarantuje tego samego silnika.

### Zachowanie typow

Rozne engines moga inaczej traktowac liczby i stringi. Celowo dobrane nieszkodliwe wyrazenie moze odroznic silniki po outputcie.

### Komunikaty bledow

Verbose errors moga ujawnic:

- nazwy engine,
- nazwy plikow szablonow,
- stack traces jezyka/runtime,
- parser tokens,
- numery linii,
- sciezki frameworka.

Bledy sa wskazowkami, nie zgoda na przyjecie wszystkich zalozen o srodowisku.

### Application stack

Headers, zachowanie frameworka, source code, package files i udokumentowana architektura moga wesprzec hipoteze o silniku. Laczymy je z runtime evidence.

## Unikaj zgadywania payloadow

Slaby workflow:

```text
try Jinja syntax
try Twig syntax
try Pug syntax
try ERB syntax
try random payload list
```

Lepszy workflow:

```text
feature
  -> controlled input
  -> reflected location
  -> suspected processing path
  -> one engine hypothesis
  -> one controlled test
  -> interpret evidence
```

## Lekcja z ukonczonego labu

W labie PortSwigger ERB poczatkowo uznalem `productId` za kontrolowany input SSTI, bo patrzylem na requesty produktowe.

Faktyczny flow:

```text
out-of-stock product
  -> redirect / request containing message parameter
  -> message rendered on the home page
```

Marker w `message` potwierdzil reflection. Dopiero test arytmetyczny ERB zwracajacy `49` potwierdzil ewaluacje.

## Co samo w sobie nie wystarcza?

- Sam HTTP method.
- Nazwa parametru brzmiaca waznie.
- HTML w odpowiedzi.
- Jeden generyczny blad `500`.
- Skopiowany payload bez rozumienia engine i kontekstu.
- Wynik narzedzia bez manualnej weryfikacji.
