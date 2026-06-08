# Podsumowanie Praktyki A06

## Trzy różne problemy designu

| Ćwiczenie | Niebezpieczna decyzja projektowa | Dowód | Wymagana zmiana designu |
|---|---|---|---|
| Cena kontrolowana przez klienta | Klient przesyłał autorytatywną wartość finansową | Zmanipulowana cena została zapisana i checkout się udał | Obliczać cenę i sumę z zaufanych danych serwerowych |
| Workflow kuponów | Reguła nie była egzekwowana w pełnym stanie promocji | Naprzemienne prawidłowe kody obniżyły zamówienie do zera | Modelować usage scope, historię i kombinacje po stronie serwera |
| Threat model platformy wydarzeń | Założenia bezpieczeństwa nie były jeszcze zapisane jako wymagania | Abuse cases ujawniły brakujące decyzje przed implementacją | Dodać jawne wymagania dotyczące autoryzacji, stanu, idempotency i granic zaufania |

## Wspólny mental model

```text
Co musi być zawsze prawdą?
  -> Jakie założenie może to naruszyć?
  -> Który aktor może podważyć założenie?
  -> Gdzie kontrola powinna być egzekwowana?
  -> Jak zweryfikujemy rezultat?
```

## Takeaway frontendowy

Frontend nadal powinien:

- ukrywać niedostępne akcje,
- walidować oczekiwany input,
- zapewniać szybki feedback,
- ograniczać przypadkowe błędy.

Backend musi niezależnie chronić:

- authorization,
- ownership,
- ceny i sumy,
- workflow state,
- private data,
- operacje wrażliwe na replay i concurrency.

## Prevention, detection i defence in depth

### Prevention

Zmienić design tak, aby niebezpieczny rezultat nie mógł wystąpić.

Przykłady:

- obliczać ceny po stronie serwera,
- egzekwować state machine,
- weryfikować ownership,
- przetwarzać webhook tylko raz.

### Detection

Zapisywać podejrzane lub odrzucone zachowanie.

Przykłady:

- powtarzana manipulacja rolą,
- replayowane webhook event IDs,
- nadmierne próby resetu.

### Defence in depth

Dodatkowo zmniejszać prawdopodobieństwo albo impact.

Przykłady:

- rate limiting,
- recent authentication,
- monitoring,
- least privilege,
- human approval.

Logging albo warning nie zastępują prevention, jeżeli workflow sam w sobie jest niebezpieczny.
