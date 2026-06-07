# Ryzyka agentów, narzędzi i RAG

## Agent i tools

Ryzyko rośnie, gdy model może robić coś więcej niż generować tekst:

- wykonywać tool calls,
- pobierać dane z systemów wewnętrznych,
- pisać do baz danych,
- wysyłać emaile,
- tworzyć pull requesty,
- uruchamiać workflow.

## RAG

RAG może wprowadzić do kontekstu niezaufane instrukcje z dokumentów. Problemem nie jest samo retrieval, tylko brak kontroli nad tym, co model może zrobić z pobraną treścią.

## Kontrole

- Retrieval musi respektować uprawnienia aktualnego użytkownika.
- Tools powinny mieć minimalne uprawnienia.
- Tool arguments muszą być walidowane poza modelem.
- Działania wysokiego ryzyka wymagają approval.
- Tool results są niezaufanym inputem dla kolejnego kroku modelu.
- Logi powinny rozróżniać proposed tool call od executed tool call.
