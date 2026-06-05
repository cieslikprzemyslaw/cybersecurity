# Threat Modeling

Notatki i mini-projekty z threat modelingu, czyli analizy zagrozen na poziomie projektu aplikacji przed implementacja albo testami.

Ten dzial pomaga cwiczyc opisywanie architektury, granic zaufania, przeplywow danych, zalozen, zagrozen, pomyslow na walidacje i mitygacji.

## Mini-projekty

| Projekt | Zakres | Glowne artefakty |
|---|---|---|
| [Simple Web Application Threat Model](simple-web-application-threat-model/README.md) | Model STRIDE dla prostej architektury: frontend React, backend API i baza danych. | [Rejestr zagrozen](simple-web-application-threat-model/threats.md), [STRIDE cheat sheet](simple-web-application-threat-model/stride-cheat-sheet.md), [Debrief](simple-web-application-threat-model/debrief.md) |

## Rola plikow

- `README.md` - indeks projektu, zakres i linki do artefaktow.
- `threats.md` - uporzadkowany rejestr zagrozen z pomyslami na walidacje i mitygacjami.
- `stride-cheat-sheet.md` - szybka sciaga STRIDE.
- `debrief.md` - refleksja po cwiczeniu, zalozenia i ograniczenia.
- `source/` - edytowalne pliki zrodlowe modelu.
- `reports/` - wyeksportowane raporty.
- `assets/` - diagramy i materialy pomocnicze.
