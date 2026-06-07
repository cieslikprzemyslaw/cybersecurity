# Deployment Controls and Least Privilege

## Główna idea

Jeśli Prompt Injection się uda, impact powinien być ograniczony przez architekturę. Model albo agent nie powinien mieć więcej uprawnień niż potrzebuje.

## Kontrole

- Oddziel konta i tokeny dla narzędzi.
- Ogranicz zakres API keys.
- Retrieval działa tylko w zakresie aktualnego użytkownika.
- Narzędzia zapisujące dane mają osobne approval gates.
- Actions wysokiego ryzyka wymagają potwierdzenia.
- Logi pokazują prompt, retrieved sources, proposed tool calls i executed tool calls.
- Sekrety nie są wkładane do prompt context bez potrzeby.

## Monitoring

Monitoruj:

- nietypowe tool calls,
- próby zmiany policy przez user/retrieved content,
- dostęp do dużej liczby dokumentów,
- output zawierający sekrety,
- rejected tool calls,
- manual approval decisions.
