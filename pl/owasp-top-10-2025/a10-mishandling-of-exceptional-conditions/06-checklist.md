# A10 Review Checklist

Używaj tylko sekcji pasujących do review danej funkcji.

## 1. Normal flow

- Co powinno normalnie się wydarzyć?
- Która operacja backendowa jest autorytatywna?
- Jaki stan powinien się zmienić?
- Jaki stan nie powinien się zmienić?

## 2. Security invariant

- Jaka reguła musi pozostać prawdziwa?
- Czy access jest denied unless authorization is confirmed?
- Czy sensitive operation może wydarzyć się tylko raz?
- Czy rekord w bazie i plik muszą istnieć razem?
- Czy audit event musi istnieć dla privileged change?

## 3. Validation and expected failures

- Missing fields?
- Empty values?
- `null` values?
- Arrays/objects where strings are expected?
- Unknown enum values?
- Malformed IDs?
- Oversized bodies?

## 4. Exceptional conditions

- Database failure?
- File storage failure?
- Authorization cannot be verified?
- Session store failure?
- Dependency timeout?
- Client disconnect?
- Two requests arrive together?

## 5. Fail-open / fail-closed

- Czy aplikacja pozwala na sensitive operation when security check fails?
- Czy access jest odmawiany, gdy authorization cannot be confirmed?
- Czy fallback obniża poziom bezpieczeństwa?
- Czy niepewność jest traktowana jak permission?

## 6. Partial state

- Które kroki zakończyły się przed błędem?
- Które zmiany zostały już zapisane?
- Czy pliki, wiadomości, emaile albo external API calls już powstały?
- Czy cleanup/rollback jest required?
- Czy użytkownik widzi stan sprzeczny z backendem?

## 7. Retry and timeout

- Czy timeout oznacza unknown outcome?
- Czy retry może tworzyć duplicate records, emails, payments, uploads albo activity logs?
- Czy frontend ślepo ponawia wrażliwe akcje?

## 8. Odpowiedzi błędów

- Czy status code jest odpowiedni?
- Czy komunikat dla klienta jest generyczny, ale zrozumiały?
- Czy request/correlation ID jest included?
- Czy stack traces, SQL queries, paths, secrets i config są ukryte?

## 9. Internal evidence

- Czy błąd jest logowany z bezpiecznym kontekstem?
- Czy log zawiera correlation ID?
- Czy repeated failures są monitorowane?
- Czy zespół wiedziałby, że cleanup/rollback failed?

## 10. Frontend behaviour

- Czy UI unika pokazywania sukcesu bez potwierdzenia?
- Czy optimistic updates są cofane albo odświeżane po błędzie?
- Czy UI rozróżnia unknown outcome od potwierdzonej porażki?
- Czy disabled button wspiera UX, a nie bezpieczeństwo?
