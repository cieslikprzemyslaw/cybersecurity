# Server-Side Request Forgery

To jest ósmy temat w serii kluczowych podatności webowych.

## Zacznij tutaj

1. [Podstawowe pojęcia](01-core-concepts.md)
2. [Ściąga: gdzie testować SSRF](02-where-to-test-cheatsheet.md)
3. [Workflow testowania SSRF](03-ssrf-testing-workflow.md)
4. [Ściąga z bypassów i filtrowania](04-bypasses-and-filtering-cheatsheet.md)
5. [Ściąga z remediacji](05-remediation-cheatsheet.md)
6. [PortSwigger Lab 01 - podstawowy SSRF na lokalny serwer](labs/portswigger-01-basic-ssrf-localhost.md)
7. [PortSwigger Lab 02 - podstawowy SSRF na inny backend](labs/portswigger-02-basic-ssrf-backend-system.md)
8. [PortSwigger Lab 03 - SSRF z filtrem opartym na blacklistcie](labs/portswigger-03-blacklist-filter-bypass.md)

## Zakres tematu

- miejsca docelowe requestów backendowych kontrolowane przez użytkownika
- dostęp do `localhost` i sieci prywatnych z kontekstu serwera
- regular vs blind SSRF
- parametry z URL-em, hostem, ścieżką albo nazwą serwisu
- bypassy blacklist, encoding, redirecty i różnice między parserami URL
- cloud metadata i ryzyko adresów link-local
- remediacja i testy regresji z perspektywy developera

## Ważna uwaga

Te notatki są tylko do legalnych labów, lokalnej praktyki i autoryzowanego testowania.

## Role plików

- `01-core-concepts.md` wyjaśnia podstawowy model myślenia o SSRF.
- `02-where-to-test-cheatsheet.md` pokazuje funkcje i parametry, w których warto szukać SSRF.
- `03-ssrf-testing-workflow.md` jest praktycznym workflow testowania black-box.
- `04-bypasses-and-filtering-cheatsheet.md` wyjaśnia typowe słabe wzorce filtrowania.
- `05-remediation-cheatsheet.md` zbiera defensywne poprawki i pomysły na regresję.
- `labs/` zawiera krótkie podsumowania legalnych labów.
