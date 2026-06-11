# Podsumowanie praktyczne A08

## Ukończone pokrycie

| Ćwiczenie | Granica zaufania | Brakująca kontrola | Wykazany rezultat |
|---|---|---|---|
| TryHackMe - świadomość deserializacji Pythona | Base64 input do deserializacji Pythona | Bezpieczny format i unikanie natywnej deserializacji niezaufanych danych | Zrozumienie na poziomie świadomości; payload został odrzucony, dlatego brak własnej deklaracji wykonania |
| PortSwigger - serializowany obiekt PHP | Session cookie do zaufanego obiektu backendu | Ochrona integralności i zaufany stan autoryzacji po stronie serwera | Privilege escalation i usunięcie innego użytkownika |

## Wspólna lekcja

```text
artefakt klienta lub źródła zewnętrznego
    -> kodowanie nie czyni go zaufanym
    -> weryfikuj przed zaufanym przetwarzaniem
```

## Co potrafię teraz sprawdzić

- właściwości sesji kontrolowane przez klienta,
- serializowane cookies,
- natywną deserializację obiektów,
- role i wartości cen generowane przez frontend,
- niepodpisane aktualizacje na poziomie świadomości,
- weryfikację artefaktów builda na poziomie świadomości,
- skrypty zewnętrzne przeglądarki i SRI,
- checksum kontra podpis,
- fail-open po błędzie weryfikacji.

## Obecne ograniczenia

Nie ukończyłem:

- zaawansowanych deserialization gadget chains,
- exploit chains Java lub .NET,
- architektury signing keys,
- pełnej implementacji SLSA/provenance,
- pełnego projektu bezpieczeństwa CI/CD.

Nie są one wymagane do obecnego statusu PASS.
