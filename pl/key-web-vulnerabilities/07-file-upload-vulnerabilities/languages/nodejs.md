# Node.js - ryzyka związane z uploadem

W aplikacjach Node.js/Express upload pliku `.js` zwykle nie powoduje jego wykonania na serwerze.

Główne ryzyka:

- path traversal przez nazwę pliku
- niebezpieczne serwowanie plików statycznych
- stored XSS through HTML/SVG
- nadpisanie pliku
- niebezpieczne parsowanie uploadowanej zawartości

## Główna lekcja

W Node.js niebezpieczny zapis, statyczne serwowanie i przetwarzanie plików są często bardziej istotne niż bezpośrednie wykonanie uploadowanego pliku.
