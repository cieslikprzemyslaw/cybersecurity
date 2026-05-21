# Cloud storage - notatki o uploadzie

Cloud storage zmienia model ryzyka. RCE może być mniej prawdopodobne, ale publiczne ujawnienie plików i stored XSS nadal mogą mieć znaczenie.

Sprawdź:

- uprawnienia bucketa lub obiektu
- publiczny dostęp
- przewidywalne URL-e
- metadane `Content-Type` kontrolowane przez użytkownika
- ryzyka związane z same-origin

## Główna lekcja

Bezpieczeństwo uploadu do cloud storage często dotyczy kontroli dostępu, metadanych, nazw obiektów i sposobu renderowania plików przez przeglądarkę.
