# Direct Prompt Injection

## Definicja

Direct Prompt Injection występuje, gdy użytkownik bezpośrednio dostarcza tekst, który próbuje zmienić zachowanie modelu albo ominąć ograniczenia aplikacji.

## Typowe cele testów

- Czy model ujawnia dane spoza uprawnień użytkownika?
- Czy user prompt może zmienić policy aplikacji?
- Czy model może wywołać tool z niedozwolonymi argumentami?
- Czy model może zmienić format outputu w sposób niebezpieczny dla downstream systemu?

## Bezpieczne testowanie

W legalnych labach zaczynaj od nieszkodliwych markerów i kontrolowanych instrukcji, które sprawdzają zmianę zachowania bez próby kradzieży danych.

Przykładowy wzorzec testu:

```text
User asks for a normal task.
User adds an instruction that conflicts with application policy.
Expected secure result: policy and authorization remain enforced outside the model.
```

## Remediacja

- Nie używaj promptu jako authorization.
- Waliduj intencję i argumenty poza modelem.
- Ogranicz narzędzia przez least privilege.
- Wymagaj approval dla działań wysokiego ryzyka.
- Loguj model output jako propozycję, nie jako potwierdzone wykonanie.
