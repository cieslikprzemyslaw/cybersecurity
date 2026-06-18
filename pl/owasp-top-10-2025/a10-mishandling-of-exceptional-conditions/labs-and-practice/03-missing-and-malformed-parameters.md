# Ćwiczenie 03: Missing and Malformed Parameters

## Scenariusz

Endpoint:

```http
PATCH /api/assessments/:assessmentId/status
```

Dozwolone wartości statusu:

```text
draft
in-progress
completed
archived
```

Reviewowany kod sprawdzał tylko:

```ts
if (!status) {
  return res.status(400).json({ error: "status is required" });
}
```

## Przypadki testowe

| Body | Oczekiwany wynik | Powód |
| --- | --- | --- |
| `{}` | reject, `400` | brak wymaganego pola |
| `{ "status": "" }` | reject, `400` | pusta wartość |
| `{ "status": null }` | reject, `400` | null nie jest poprawnym statusem tekstowym |
| `{ "status": ["completed"] }` | reject, `400` | zły typ danych |
| `{ "status": "admin" }` | reject, `400` | nieznana wartość enum |
| `{ "status": "completed" }` | accept, `200`, tylko jeśli auth/authz i state transition rules przechodzą | poprawna wartość inputu |

## Co poprawiłem

Na początku potraktowałem brak authorization check jako możliwe `500`. Poprawiony model:

```text
brak auth/authz check -> zwykle A01 Broken Access Control albo A06 Insecure Design
security dependency istnieje, ale pada i aplikacja pozwala na akcję -> A10 fail-open
```

Poprawny request od niezalogowanego użytkownika nie powinien dawać `500`.

- Brak logowania -> `401 Unauthorized`
- Zalogowany, ale bez uprawnień -> `403 Forbidden` albo celowe `404 Not Found`
- Valid i allowed -> `200 OK`
- Unexpected server failure -> kontrolowane `500`

## Lekcja A10

Missing, null, empty, wrong-type albo unknown-enum values powinny być odrzucone jako expected validation failures. Nie powinny wejść do business logic i spowodować dziwnych exceptions, unsafe state transitions albo nieoczekiwanego zachowania serwera.

## Bezpieczne oczekiwane zachowanie

Backend powinien:

- zwalidować shape body,
- zwalidować, że `status` jest stringiem,
- zwalidować, że `status` jest jedną z dozwolonych wartości,
- odrzucać unknown fields, jeśli kontrakt API wymaga strict input,
- sprawdzić authentication,
- sprawdzić authorization dla konkretnego assessmentu,
- sprawdzić, czy state transition jest dozwolony,
- zwrócić kontrolowane `400`, `401`, `403`/`404` albo `200`, zależnie od przypadku.

## Testy regresji

Przydatne testy:

- `{}` zwraca `400`,
- pusty string zwraca `400`,
- `null` zwraca `400`,
- array value zwraca `400`,
- object value zwraca `400`,
- unknown enum zwraca `400`,
- valid enum bez zalogowania zwraca `401`,
- valid enum bez uprawnień zwraca `403` albo celowe `404`,
- valid enum z uprawnionym userem zmienia status,
- malformed input nigdy nie zmienia assessment status.

## Frontend takeaway

Frontend form validation jest przydatny, ale backend musi egzekwować API contract. Atakujący i testerzy mogą ominąć UI przez Burp, ZAP, Postman, DevTools albo bezpośrednie skrypty.
