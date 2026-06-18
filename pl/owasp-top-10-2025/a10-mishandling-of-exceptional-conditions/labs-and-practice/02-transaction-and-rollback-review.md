# Ćwiczenie 02: Transaction and Rollback Review

## Scenariusz

Normalny flow uploadu pliku evidence:

```text
1. Utwórz rekord evidence w bazie.
2. Zapisz uploadowany plik w storage.
3. Zaktualizuj assessment evidence count.
4. Zwróć 201 Created.
```

Scenariusz awarii:

```text
1. Rekord evidence został utworzony.
2. Plik został zapisany.
3. Aktualizacja assessment evidence count padła.
4. API zwróciło 500.
```

## Jaki stan się zmienił

Awaria nie oznacza, że nic się nie stało.

Stan, który mógł się już zmienić:

- może istnieć rekord evidence w bazie,
- może istnieć uploadowany plik,
- licznik assessmentu może zostać bez zmiany,
- UI może nie wiedzieć, czy evidence zostało dodane.

To jest problem partial state:

```text
evidence exists + file exists + assessment count failed
```

## Co poprawiłem

Na początku skupiłem się tylko na file storage. Poprawiony widok jest szerszy:

```text
Które części operacji już się udały?
Która część padła?
Czy system może wrócić do jednego znanego, bezpiecznego stanu?
```

Na tym poziomie nie muszę projektować idealnej architektury storage. Muszę rozpoznać, że nieukończona operacja może zostawić niespójne dane, pliki albo liczniki.

## Lekcja A10

Odpowiedź `500` nie dowodzi rollbacku. Mówi tylko frontendowi, że API nie zwróciło sukcesu.

Backend nie powinien zostawić normalnie dostępnego evidence item, jeżeli cała logiczna operacja nie została ukończona.

## Bezpieczne oczekiwane zachowanie

Backend powinien:

- zakończyć całą logiczną operację sukcesem,
- albo cofnąć zmiany w bazie tam, gdzie to możliwe,
- posprzątać albo odizolować częściowo zapisane pliki,
- nie oznaczać operacji jako success, gdy wymagany krok padł,
- zwrócić kontrolowany błąd,
- logować wystarczające evidence do investigation.

Na moim obecnym poziomie proste wymaganie brzmi:

```text
Po nieudanym evidence upload assessment nie powinien zawierać aktywnego, niespójnego evidence item.
```

## Testy regresji

Przydatne testy:

- zasymuluj awarię podczas aktualizacji `evidenceCount`,
- sprawdź, że API zwraca kontrolowane `500`,
- sprawdź, że evidence record nie zostaje jako aktywny/używalny, jeśli pełna operacja padła,
- sprawdź, że uploadowany plik nie zostaje jako normalnie dostępny attachment bez valid DB record,
- sprawdź, że assessment zostaje w poprzednim bezpiecznym stanie,
- sprawdź, że error response nie ujawnia internal details,
- sprawdź, że retry nie tworzy duplicate evidence.

## Frontend takeaway

Frontend nie powinien zakładać, że `500` oznacza, że nic nie zostało zapisane. Powinien unikać pokazywania potwierdzonego sukcesu i może potrzebować odświeżyć evidence list albo pokazać stan "unable to confirm".
