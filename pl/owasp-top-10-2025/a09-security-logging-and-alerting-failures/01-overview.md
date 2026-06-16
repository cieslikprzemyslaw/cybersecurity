# A09 — Overview

## Definicja

Security Logging and Alerting Failures występuje wtedy, gdy aplikacja lub organizacja nie potrafi skutecznie utworzyć, zachować, przeanalizować, wykryć, eskalować albo obsłużyć zdarzeń istotnych dla bezpieczeństwa.

Typowe problemy:

- ważne zdarzenia nie są logowane,
- eventy są niespójne albo nie zawierają kontekstu potrzebnego do dochodzenia,
- sekrety lub zbędne dane osobowe trafiają do logów,
- niezaufane dane mogą sfałszować albo uszkodzić wpis,
- logi mogą zostać odczytane, zmienione lub usunięte przez nieuprawnione osoby,
- zdarzenia są przechowywane wyłącznie lokalnie i znikają podczas awarii lub incydentu,
- podejrzane wzorce nie są monitorowane ani korelowane,
- alert nie ma sensownego progu, kontekstu, właściciela lub playbooka,
- nadmiar false positives ukrywa prawdziwe ataki,
- awaria logowania lub alertowania powoduje, że główna kontrola bezpieczeństwa działa fail open.

A09 pozostaje na pozycji dziewiątej w OWASP Top 10 2025. Nazwa mocniej podkreśla alerting, ponieważ samo zapisanie zdarzenia nie wywołuje działania podczas aktywnego incydentu.

## Ważne rozróżnienia

### Application log

Wpis tworzony przez aplikację o jej działaniu, stanie, błędach albo operacjach.

### Security log

Wpis istotny dla detekcji, accountability, investigation albo działania kontroli bezpieczeństwa.

### Audit log

Chronologiczny zapis wrażliwej operacji, zwykle obejmujący aktora, cel, zmianę, czas i rezultat.

### Debug log

Szczegółowe dane do troubleshootingu. Debug output może zawierać stack trace, wewnętrzne ścieżki, konfigurację, request data albo sekrety i nie powinien być bezmyślnie włączony na produkcji.

### Monitoring

Zbieranie, obserwowanie, analizowanie i korelowanie logów, metryk i eventów w czasie.

### Alerting

Powiadomienie albo automatyczna akcja wywołana po spełnieniu zdefiniowanego warunku detekcji.

### Incident response

Triage, investigation, containment, remediation, recovery, zachowanie dowodów i wyciągnięcie wniosków po wykryciu incydentu.

## A09 to nie tylko „brak logów”

Kategoria obejmuje cały łańcuch:

```text
required event
    -> correct and safe event structure
    -> trustworthy storage
    -> monitoring and correlation
    -> actionable alert
    -> owned response
```

Awaria dowolnego etapu może stworzyć visibility gap, detection gap albo response gap.

## Fakty, założenia i luki

### Fakty

- jaki event rzeczywiście powstał,
- jakie pola zawierał,
- gdzie został zapisany,
- czy reguła została dopasowana,
- czy alert został dostarczony,
- kto go otrzymał,
- jaka reakcja rzeczywiście nastąpiła.

### Założenia

- „platforma pewnie to loguje”,
- „security team pewnie to monitoruje”,
- „alert na pewno został dostarczony”,
- „adres IP dowodzi, kto wykonał akcję”.

### Detection gap

Podejrzany lub złośliwy wzorzec może wystąpić bez reguły, która go rozpozna.

### Response gap

Alert istnieje, ale brak właściciela, ścieżki eskalacji albo aktualnego playbooka powoduje, że reakcja nie następuje na czas.

## Powiązane klasy słabości

A09 obejmuje między innymi:

- nieprawidłowe neutralizowanie outputu do logów,
- pomijanie informacji istotnych dla bezpieczeństwa,
- zapisywanie danych wrażliwych w logach,
- niewystarczające logowanie.

## Wpływ

Wpływ jest często pośredni, ale poważny:

- account takeover albo privilege abuse pozostają niezauważone,
- atakujący może działać dłużej,
- nie da się odtworzyć pełnego zakresu incydentu,
- dowody są niekompletne lub niewiarygodne,
- alerty są opóźnione albo ignorowane,
- dane osobowe lub sekrety wyciekają przez system logowania,
- rosną skutki operacyjne i regulacyjne.

Logging nie zapobiega pierwotnej podatności. Zapewnia visibility, detection, accountability, investigation i response.

## Odpowiedź w stylu rozmowy rekrutacyjnej

> A09 to awaria cyklu życia security eventu. Aplikacja może pominąć zdarzenie, zapisać niebezpieczne lub niepełne dane, nie chronić logu albo zapisać event bez detekcji i alertu. Podczas review sprawdziłbym, gdzie backend podejmuje autorytatywną decyzję, jaki structured event emituje, jakie sekrety wyklucza, jaka reguła wykrywa nadużycie, kto jest właścicielem alertu oraz jak testowany jest pełny przepływ.
