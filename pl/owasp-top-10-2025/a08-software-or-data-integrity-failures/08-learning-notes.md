# Learning Notes

## Początkowe rozumienie

Poprawnie rozpoznałem, że dane z przeglądarki nie powinny być traktowane jako autorytatywne. Rozumiałem również, że Base64 nie zapewnia bezpieczeństwa.

Moje początkowe sformułowania były jednak zbyt szerokie:

- opisałem artefakt tylko jako cookie, zamiast serializowanego obiektu sesji wewnątrz cookie,
- pomieszałem właściwości checksum z właściwościami podpisu cyfrowego,
- opisałem szyfrowanie jako ochronę przed modyfikacją bez wyraźnego zaznaczenia potrzeby uwierzytelnienia.

## Korekty

### Cookie kontra chroniony artefakt

Precyzyjniejsze zdanie:

> Kontrolowanym artefaktem był serializowany obiekt PHP `User` przechowywany w session cookie zakodowanym przez URL i Base64.

### Checksum kontra podpis cyfrowy

Poprawne rozróżnienie:

```text
checksum
    -> czy zawartość odpowiada oczekiwanemu hash?

digital signature
    -> czy zawartość pozostała niezmieniona
       i została podpisana oczekiwanym zaufanym kluczem?

encryption
    -> czy nieuprawniona osoba może odczytać zawartość?
```

Checksum sam z siebie nie potwierdza autora artefaktu.

### Szyfrowanie i modyfikacja

Szyfrowanie chroni przede wszystkim poufność. Integralność i autentyczność wymagają MAC, podpisu albo authenticated encryption.

## Proces nauki TryHackMe

Zadanie Python wprowadziło `pickle` i `__reduce__`.

Próbowałem wygenerować Base64 pickle payload, ale aplikacja zwróciła:

```text
UnpicklingError: invalid load key
```

Nie otrzymałem wiarygodnego dowodu, że mój payload wykonał się na serwerze. Nie deklaruję więc file read ani code execution na podstawie tej próby.

Użyteczna lekcja była architektoniczna:

```text
niezaufany serializowany input
    -> pickle deserialization
    -> możliwe zachowanie podczas rekonstrukcji
```

Główna kontrola to unikanie unpickling niezaufanego inputu, a nie próba sanitizacji każdego niebezpiecznego obiektu po rekonstrukcji.

## Rozumowanie w PortSwigger

### Obserwacja

Session cookie po odkodowaniu:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

### Hipoteza

Jeśli backend ufa polu `admin` kontrolowanemu przez klienta, zmiana `b:0` na `b:1` powinna dać zachowanie administracyjne.

### Kontrolowany test

Zmieniłem wyłącznie boolean. Nazwa obiektu, długości stringów, username i początkowa ścieżka requestu pozostały takie same.

### Dowody

Zmiana spowodowała:

1. pojawienie się linku admin,
2. zwrócenie panelu przez `/admin`,
3. wykonanie `/admin/delete?username=carlos`.

### Wniosek

Backend zaufał zmienionemu stanowi obiektu kontrolowanemu przez klienta. Udowodniony wpływ to privilege escalation i nieautoryzowana akcja administracyjna.

Nie deklaruję RCE, ponieważ nie było na to dowodów.

## Rozumowanie remediation

Mój pierwszy pomysł zawierał dwie poprawne kontrole:

1. backend powinien korzystać z własnych zaufanych danych zamiast przyjmować stan frontendu,
2. stan klienta wymagający zaufania powinien mieć ochronę kryptograficzną.

Lepszy projekt:

```text
losowy opaque session ID w cookie
    -> server-side session lookup
    -> rola z zaufanego session store lub bazy
    -> autoryzacja na każdym chronionym endpoincie
```

Podpisanie danych klienta może wykryć modyfikację, ale nie usuwa potrzeby autoryzacji po stronie serwera. Nadal należy unikać natywnej deserializacji niezaufanych obiektów.

## A03 kontra A08

Dla złośliwej paczki npm zacząłbym od A03, ponieważ dotyczy szerszego procesu zależności i supply chain.

A08 zastosowałbym do:

- niepodpisanej aktualizacji,
- zmienionego artefaktu builda zaakceptowanego do deploymentu,
- nieweryfikowanego skryptu zewnętrznego,
- zmienionego serializowanego obiektu zaakceptowanego przez backend.

## Końcowy wniosek

> A08 nie dotyczy tego, czy dane są trudne do odczytania. Dotyczy tego, czy zaufany komponent posiada dowód, że software lub dane pochodzą z oczekiwanego źródła, pozostały niezmienione, nadal są ważne i są autoryzowane do aktualnego użycia.
