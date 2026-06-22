# OWASP Top 10 2025 — podsumowanie nauki

Ta notatka zbiera najważniejsze wnioski z mapowania OWASP Top 10 2025. [README](README.md) zostaje indeksem katalogu, a ta strona jest krótkim podsumowaniem i refleksją po całym sprincie.

Sprint był zbudowany wokół prostego celu: przejść od teorii do powtarzalnych nawyków review. Dla każdej kategorii chciałem rozumieć przyczynę źródłową, ścieżkę danych kontrolowanych przez użytkownika, punkt egzekwowania reguły po stronie backendu, bezpieczny sposób testowania, wpływ podatności oraz test regresji po poprawce.

## Co objął sprint

Przerobiłem wszystkie kategorie OWASP Top 10 2025:

- A01 Broken Access Control: błędy autoryzacji, myślenie w stylu IDOR i sprawdzanie granic zaufania.
- A02 Security Misconfiguration: ujawnione dane debugowe, zbyt szczegółowe błędy, niebezpieczne ustawienia domyślne i konfiguracja zależna od środowiska.
- A03 Software Supply Chain Failures: zaufanie do zależności, lifecycle scripts, lockfile, SBOM, integralność CI/CD i ryzykowne sposoby instalacji pakietów.
- A04 Cryptographic Failures: obsługa danych wrażliwych, słaby projekt tokenów, różnica między kodowaniem a szyfrowaniem, hashowanie i bezpieczeństwo cookies.
- A05 Injection: SQL Injection, NoSQL Injection, OS Command Injection, SSTI, mapowanie XSS i świadomość prompt injection.
- A06 Insecure Design: błędy logiki biznesowej, niebezpieczne założenia, abuse cases, threat modelling i review STRIDE.
- A07 Authentication Failures: logika resetowania hasła, egzekwowanie MFA, stan sesji, enumeration i cykl życia uwierzytelniania.
- A08 Software or Data Integrity Failures: integralność danych, aktualizacji, paczek, artefaktów, buildów i zaufanych workflow.
- A09 Security Logging and Alerting Failures: zdarzenia bezpieczeństwa, audit logi, alertowanie, bezpieczeństwo logów, wartość detekcyjna i gotowość do reakcji na incydenty.
- A10 Mishandling of Exceptional Conditions: obsługa błędów, fail-open, niebezpieczne fallbacki, retry, timeouty i niepoprawne stany aplikacji.

## Powtarzalny wzorzec review

Najbardziej użyteczny nawyk polegał na zadawaniu tych samych pytań przy każdej analizie:

- Co kontroluje użytkownik?
- Gdzie trafiają te dane?
- Który komponent im ufa?
- Czy regułę egzekwuje backend, czy frontend tylko sugeruje poprawne zachowanie?
- Co się stanie, gdy zawiedzie walidacja, zależność, sesja albo zewnętrzna usługa?
- Czy system zatrzyma się bezpiecznie, czy będzie kontynuował w niebezpiecznym stanie?
- Co powinno zostać zalogowane, zaalertowane i przetestowane po poprawce?

Ten wzorzec połączył osobne kategorie w jeden praktyczny sposób pracy. Ułatwił też opisywanie wpływu prostym językiem i przerabianie poprawki na test regresji.

## Najważniejsze wnioski

Sprint wzmocnił kilka tematów, które wracały w wielu kategoriach:

- kontrolki frontendowe są ważne dla UX, ale decyzje bezpieczeństwa muszą być egzekwowane po stronie backendu,
- input trzeba obsługiwać zależnie od kontekstu, a nie jedną ogólną metodą „czyszczenia”,
- authentication i authorization to osobne problemy i trzeba je sprawdzać oddzielnie,
- bezpieczny design ma znaczenie jeszcze przed implementacją,
- logi są zasobem bezpieczeństwa, a nie tylko miejscem do debugowania,
- zależności, build i CI/CD są częścią bezpieczeństwa aplikacji,
- obsługa błędów może tworzyć ryzyko, jeżeli aplikacja pozwala kontynuować operację w niebezpiecznym stanie,
- testy regresji są częścią poprawki, a nie dodatkiem „na później”.

## Co dalej

Notatki są celowo defensywne i pisane z perspektywy developerskiej. To nie jest exploit guide i nie zawiera prawdziwych danych celów, credentials, prywatnych informacji ani instrukcji testowania systemów bez zgody.

Następne obszary do rozwinięcia to:

1. Docker i container security.
2. CI/CD security.
3. Security review lokalnych workflow developerskich.
4. Bardziej realistyczne security finding write-ups.
5. Dalsza praktyka z secure design i threat modelling.
