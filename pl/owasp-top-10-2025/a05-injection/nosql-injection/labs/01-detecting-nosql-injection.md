# Lab: Detecting NoSQL Injection

> PortSwigger Web Security Academy: Detecting NoSQL injection

## Cel

Wykryć NoSQL Injection w legalnym labie i potwierdzić, że input użytkownika zmienia zachowanie query.

## Punkt wejścia

Testowany był parametr używany przez funkcję filtrowania/lookupu.

Najpierw potrzebny był baseline:

- normalny request,
- normalna odpowiedź,
- oczekiwany zestaw wyników.

## Test apostrofem

Dodanie pojedynczego apostrofu spowodowało zmianę odpowiedzi.

To sugerowało, że input nie jest traktowany wyłącznie jako zwykły tekst i może trafiać do składni query.

## Test true/false

Kolejny krok to porównanie kontrolowanych warunków:

```text
warunek zawsze prawdziwy
warunek zawsze fałszywy
```

Jeżeli odpowiedzi różnią się stabilnie, można uznać, że input wpływa na logikę query.

## Wniosek techniczny

Dowody wskazywały na Syntax Injection, a nie Operator Injection.

Powód:

- test nie wysyłał zagnieżdżonego obiektu typu `$ne` albo `$regex`,
- input przełamywał kontekst stringa/wyrażenia,
- dodana logika true/false wpływała na wynik.

## Impact w labie

Warunek always-true mógł zmienić zestaw zwracanych rekordów i ujawnić dane spoza zamierzonego filtra.

## Remediacja

- Nie doklejać inputu użytkownika do custom query expressions.
- Unikać `$where` i JavaScript-style query construction.
- Budować query z pól i operatorów wybranych po stronie serwera.
- Walidować typy i długość inputu.
- Nie ujawniać szczegółowych błędów bazy ani interpretera.

## Test regresji

Po poprawce:

- apostrof powinien być traktowany jako dane albo odrzucony kontrolowanie,
- warunek always-true nie powinien rozszerzać wyników,
- warunek always-false nie powinien dawać stabilnego oracle,
- błędy powinny być generyczne.
