# LLM01 Prompt Injection Awareness Checklist

- [ ] Czy funkcja używa LLM, agenta, RAG albo external-content ingestion?
- [ ] Czy system/developer instructions są oddzielone od inputu użytkownika?
- [ ] Czy retrieved content jest traktowany jako niezaufane dane?
- [ ] Czy access control jest egzekwowany przed retrieval?
- [ ] Czy tools mają least privilege?
- [ ] Czy tool arguments są walidowane poza modelem?
- [ ] Czy high-risk actions wymagają human approval?
- [ ] Czy model output jest traktowany jako niezaufany przed downstream use?
- [ ] Czy testy obejmują direct i indirect Prompt Injection?
- [ ] Czy raportowanie oddziela model output od potwierdzonego wykonania i impactu?
