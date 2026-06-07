# LLM01 Learning Notes

## Najważniejszy model

Prompt Injection to nie tylko "model dał się namówić". W AppSec liczy się to, czy niezaufana treść mogła wpłynąć na dane, narzędzia, decyzje albo działania aplikacji.

## Najważniejsze lekcje

- Model nie jest security boundary.
- Prompt wording pomaga, ale nie zastępuje access control.
- Retrieved content jest niezaufanym inputem.
- Model output nie dowodzi wykonania.
- Tool use musi być walidowany i autoryzowany poza modelem.
- Least privilege ogranicza impact.
- Dowody muszą pokazywać realny skutek, nie tylko deklarację modelu.

## Co powinienem umieć wyjaśnić

- różnicę między direct i indirect Prompt Injection,
- dlaczego RAG zwiększa powierzchnię ataku,
- dlaczego system prompt hardening jest defence in depth,
- jak ograniczać tools i retrieval,
- jak pisać testy regresji dla agentów,
- jak odróżnić proposed action od executed action.
