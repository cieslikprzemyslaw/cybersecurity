# A09 — Checklista Review

Użyj sekcji istotnych dla analizowanej funkcji. Nie dodawaj eventów bez jasnego celu bezpieczeństwa.

## 1. Asset i akcja

- Jaka tożsamość, permission, data, transaction albo workflow mają znaczenie?
- Które akcje są high value, privileged, irreversible lub sensitive?
- Jaki abuse case powinien stać się widoczny?

## 2. Security-relevant events

- Czy logowane są ważne sukcesy i niepowodzenia?
- Czy pokryto authentication, MFA, session i recovery?
- Czy authorization denials i privileged actions są widoczne?
- Czy zmiany ról i uprawnień mają audit trail?
- Czy sensitive create/read/update/delete/export jest logowane tam, gdzie jest to uzasadnione?
- Czy pokryto input validation, token validation, upload scan i rate limiting?
- Czy workflow bypass i out-of-sequence actions są widoczne?

## 3. Kontekst eventu

- Czy event odpowiada na when, where, who, what i result?
- Czy istnieje stabilny internal actor ID?
- Czy target resource albo target user jest wskazany?
- Czy `eventName`, `result` i `severity` są osobnymi polami?
- Czy użyto bezpiecznego reason code zamiast niekontrolowanego error text?
- Czy istnieje request, correlation albo interaction ID?
- Czy timestampy i UTC offsets są spójne?
- Czy nazwy pól, typy danych i długości są udokumentowane?

## 4. Sensitive data

- Czy hasła są wykluczone?
- Czy session cookies i session IDs są wykluczone?
- Czy access, refresh, reset i MFA tokens są wykluczone?
- Czy API keys i cryptographic keys są wykluczone?
- Czy `Authorization` headers są wykluczone?
- Czy uniknięto pełnych request i response bodies?
- Czy PII, health, payment i CMS data są minimalizowane?
- Czy review obejmuje third-party telemetry i retention?
- Czy masking, hashing lub pseudonymisation są rzeczywiście potrzebne i bezpieczne?

## 5. Event ownership

- Który komponent egzekwuje security control?
- Czy ten komponent emituje autorytatywny event?
- Czy browser, WAF, database i infrastructure logs są traktowane jako supporting sources tam, gdzie to właściwe?
- Czy użytkownik może zablokować albo sfabrykować jedyny dostępny dowód?

## 6. Log injection

- Które pola są user-controlled?
- Czy użyto structured logging?
- Czy newline lub control characters mogą zmienić strukturę?
- Czy użytkownik może kontrolować event name, severity, result lub reason code?
- Czy zdefiniowano długości i obsługę znaków?
- Czy downstream parsery i viewery zostały przetestowane?

## 7. Collection i protection

- Gdzie logi są zbierane?
- Czy security logs są centralizowane albo niezawodnie forwardowane?
- Czy są chronione in transit i at rest?
- Kto może je odczytać, zmienić lub usunąć?
- Czy write permissions są oddzielone od read/admin access?
- Czy zatrzymanie logowania, tampering albo deletion można wykryć?
- Czy retention wystarcza do opóźnionego dochodzenia i respektuje privacy requirements?
- Czy zdefiniowano backup i disposal?

## 8. Detection logic

- Jaki dokładny attack albo abuse pattern ma zostać wykryty?
- Jakich eventów i correlation keys potrzeba?
- Jaki threshold i time window są odpowiednie?
- Czy reguła tworzy oczywiste false positives?
- Czy low-and-slow albo distributed behaviour może ją ominąć?
- Czy success po wcześniejszych failures otrzymuje wyższy priorytet?
- Czy DAST i kontrolowane security tests powinny uruchomić regułę?

## 9. Alerting

- Czy alert jest actionable i contextual?
- Czy ma severity i confidence?
- Czy istnieje nazwany owner lub on-call route?
- Czy ścieżka eskalacji jest udokumentowana?
- Czy deduplication albo suppression ograniczają alert fatigue?
- Czy monitorowana jest dostawa alertu?

## 10. Response

- Co powinno się wydarzyć po high-confidence alert?
- Czy uwzględniono evidence preservation, containment, session invalidation i account protection?
- Czy reakcja jest proporcjonalna do confidence?
- Czy playbook istnieje i jest aktualny?
- Czy prevention, detection, response i evidence preservation są rozdzielone?

## 11. Failure handling

- Czy awaria loggera, collectora albo kanału alertów powoduje fail open głównej kontroli?
- Czy logging failure może wyczerpać zasoby albo zablokować legalny ruch?
- Czy utrata storage, uprawnień, sieci albo ingestion jest obsługiwana bezpiecznie?
- Czy istnieje monitoring dropped lub delayed events?

## 12. Evidence z weryfikacji

- Czy test dowodzi, że event został wyemitowany?
- Czy dowodzi, że sekrety zostały wykluczone?
- Czy dowodzi, że collection i integrity controls zadziałały?
- Czy dowodzi dopasowania reguły?
- Czy dowodzi dostarczenia alertu do właściwego ownera?
- Czy dowodzi, że playbook można wykonać?
