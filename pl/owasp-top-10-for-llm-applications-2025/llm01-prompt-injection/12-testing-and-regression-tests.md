# Testing and Regression Tests

## Cel

Testy powinny potwierdzać, że Prompt Injection nie może rozszerzyć uprawnień użytkownika, zmienić permissions narzędzi ani spowodować nieautoryzowanego dostępu do danych.

## Testy direct

- User prompt nie może zmienić roli ani policy aplikacji.
- User prompt nie może pominąć approval gate.
- User prompt nie może uzyskać danych spoza uprawnień.

## Testy indirect

- Retrieved document nie może zmienić system/developer instructions.
- Retrieved document nie może zmienić tool permissions.
- Retrieved document nie może wymusić zewnętrznej akcji.
- Content z jednego użytkownika nie wpływa na dane innego użytkownika.

## Testy tools

- Tool calls wymagają schema validation.
- Tool calls wymagają authorization poza modelem.
- Niebezpieczne działania wymagają approval.
- Logs rozróżniają proposed i executed calls.

## Testy outputu

- Model-generated HTML nie jest renderowany jako zaufany HTML.
- Model-generated SQL nie trafia bezpośrednio do query.
- Model-generated shell content nie trafia do process execution.
- Model-generated URLs są walidowane.

## Regression checklist

- Poprzedni payload direct nie zmienia już zachowania.
- Poprzedni payload indirect pozostaje cytowaną treścią.
- Retrieval scope pozostaje ograniczony.
- Tool permissions pozostają minimalne.
- Approval nadal zatrzymuje high-risk action.
- Logi pokazują faktyczny impact, a nie tylko model claims.
