# Checklista bezpieczeństwa CI/CD

Użyj tej checklisty przy review projektu opartego o GitHub Actions.

## Uprawnienia workflowa

- [ ] Czy workflow ma jawnie ustawione uprawnienia?
- [ ] Czy joby walidacyjne używają tylko odczytu, gdy to wystarcza?
- [ ] Czy uprawnienia zapisu są ograniczone do jobów, które naprawdę ich potrzebują?
- [ ] Czy uprawnienia do wdrożeń są oddzielone od walidacji pull requestów?

Dobry punkt startowy:

```yaml
permissions:
  contents: read
```

## Stabilność workflowa

- [ ] Czy każdy job ma timeout?
- [ ] Czy workflow używa concurrency, żeby anulować stare uruchomienia?
- [ ] Czy przy błędzie runtime dostępne są logi?
- [ ] Czy cleanup działa z `if: always()` tam, gdzie jest potrzebny?

## Walidacja pull requesta

- [ ] Czy CI uruchamia się dla pull requestów do chronionej gałęzi?
- [ ] Czy CI uruchamia się po merge/push do chronionej gałęzi?
- [ ] Czy unikamy podwójnych uruchomień dla branchy z PR?
- [ ] Czy wymagane checki są czytelne?

## Build i sprawdzenie działania

- [ ] Czy projekt uruchamia format/lint/typecheck/testy/build?
- [ ] Czy warstwa bazy albo schematu ma jawną walidację?
- [ ] Jeżeli używany jest Docker, czy CI buduje obraz?
- [ ] Czy CI uruchamia kontener?
- [ ] Czy CI sprawdza health endpoint?
- [ ] Czy zmienne środowiskowe dla runtime są jawne?

## Bezpieczeństwo zależności

- [ ] Czy skanowanie zależności jest włączone?
- [ ] Czy podatności high/critical blokują PR?
- [ ] Czy istnieje polityka tymczasowej akceptacji ryzyka?
- [ ] Czy zaakceptowane ryzyko wymaga powodu?
- [ ] Czy zaakceptowane ryzyko ma datę wygaśnięcia?
- [ ] Czy zaakceptowane ryzyko ma follow-up issue?
- [ ] Czy najpierw próbujemy naprawić direct dependency, zanim użyjemy override?

## SAST

- [ ] Czy analiza statyczna jest włączona dla używanego stacku?
- [ ] Czy wyniki są sprawdzane, a nie ignorowane?
- [ ] Czy pierwszy etap skupia się na widoczności i triage?
- [ ] Czy jest plan na potwierdzone problemy?
- [ ] Czy false positive są dokumentowane ostrożnie?

## Ochrona repozytorium

- [ ] Czy direct push do chronionej gałęzi jest zablokowany?
- [ ] Czy merge wymaga pull requesta?
- [ ] Czy wymagane checki muszą przejść przed merge?
- [ ] Czy administratorzy mogą ominąć ochronę?
- [ ] Czy ustawienie bypass jest świadomą decyzją?

## Higiena sekretów

- [ ] Czy `.env` jest ignorowany?
- [ ] Czy lokalne pliki bazy danych są ignorowane?
- [ ] Czy logi są ignorowane?
- [ ] Czy uploady i dane runtime są ignorowane?
- [ ] Czy lokalne sekrety Dockera są ignorowane?
- [ ] Czy w repozytorium nie ma prawdziwych sekretów?
- [ ] Czy workflowy nie używają sekretów bez potrzeby?
- [ ] Czy sekrety nie są wypisywane do logów?

## Końcowe pytanie review

```text
Jeżeli ryzykowna zmiana trafi do tego repozytorium, która kontrola powinna ją złapać?
```
