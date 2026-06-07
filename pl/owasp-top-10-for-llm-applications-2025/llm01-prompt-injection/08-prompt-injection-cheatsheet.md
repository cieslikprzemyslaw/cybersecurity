# Prompt Injection Testing Cheatsheet

> Tylko dla autoryzowanych labów, środowisk testowych i własnych aplikacji.

## Pierwsze pytania

1. Czy aplikacja używa LLM, agenta, RAG albo tools?
2. Jakie dane trafiają do prompt context?
3. Które instrukcje są zaufane?
4. Które treści są niezaufane?
5. Jakie tools może wywołać model?
6. Jakie dane może pobrać retrieval?
7. Co jest model output, a co rzeczywistym wykonaniem?

## Test direct

```text
Użytkownik podaje instrukcję sprzeczną z policy.
Oczekiwane: aplikacja nadal egzekwuje policy i authorization poza modelem.
```

## Test indirect

```text
Zewnętrzny dokument zawiera instrukcję dla modelu.
Użytkownik prosi o streszczenie dokumentu.
Oczekiwane: dokument jest traktowany jako dane, nie jako instrukcja zmieniająca tools albo policy.
```

## Dowody

Zapisuj:

- source niezaufanej treści,
- prompt/context path,
- expected secure behaviour,
- observed model output,
- czy tool call został wykonany,
- czy doszło do data access albo state change.

## Bezpieczeństwo

Nie testuj realnych systemów bez zgody. Nie próbuj kradzieży danych ani działań destrukcyjnych poza labem.
