# AI Prompt Injection Awareness

## Decyzja mapowania

Prompt Injection jest ryzykiem w stylu injection, ale nie jest tutaj opisane jako pełny klasyczny temat web-injection.

OWASP Top 10:2025 A05 Injection wskazuje, że podobna klasa injection istnieje w aplikacjach LLM i odsyła do **OWASP LLM01:2025 Prompt Injection** jako dedykowanego źródła.

Pełne notatki:

```text
pl/owasp-top-10-for-llm-applications-2025/
  llm01-prompt-injection/
```

## Dlaczego jest powiązane z A05

Wspólna idea wysokiego poziomu: niezaufany input zmienia sposób, w jaki interpreter przetwarza zadanie.

```text
Classic injection:
untrusted input → browser/database/shell/template interpreter → unintended execution

Prompt Injection:
untrusted content → LLM or agent → unintended instruction-following behaviour
```

## Ważna różnica

SQL, OS command i wiele template injections przekracza deterministyczną granicę gramatyki albo kod/dane. Prompt Injection działa przez probabilistyczną interpretację natural-language context. Impact zależy też mocno od aplikacji, podłączonych danych, tools i uprawnień.

## Punkty awareness dla AppSec

- traktuj model jako niezaufany i niedeterministyczny,
- oddziel trusted instructions od user i retrieved content,
- traktuj external content jako niezaufane dane,
- nigdy nie używaj promptu jako authorization,
- egzekwuj access control przed retrieval,
- ogranicz tools przez least privilege,
- waliduj tool arguments przed wykonaniem,
- traktuj model output jako niezaufany input,
- wymagaj approval dla high-risk actions,
- odróżniaj generated output od realnego execution i impact.

## References

- [OWASP A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)
- [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
