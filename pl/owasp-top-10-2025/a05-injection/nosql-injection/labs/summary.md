# NoSQL Injection Lab Summary

## Ukończone laby

1. Detecting NoSQL injection
2. Exploiting NoSQL injection to extract data

## Porównanie

| Obszar | Detecting NoSQL injection | Extracting data |
| --- | --- | --- |
| Główny cel | Potwierdzić injection | Użyć oracle do ekstrakcji sekretu |
| Forma | Syntax Injection | Syntax Injection |
| Dowód | Różne odpowiedzi dla true/false | Stabilny marker true/false |
| Impact | Zmieniony zestaw wyników | Odzyskanie hasła administratora w labie |
| Najważniejsza lekcja | Apostrof i warunki kontrolowane | Background endpoint i oracle |

## Wspólny model

```text
kontrolowany input
-> query expression
-> warunek true/false
-> różnica odpowiedzi
-> dowód wpływu na query
```

## Różnica względem Operator Injection

W tych labach nie chodziło o wysłanie zagnieżdżonego obiektu operatora typu `$ne`.

Dowody wskazywały na Syntax Injection:

- input przełamywał kontekst wyrażenia,
- warunki JavaScript-style zmieniały odpowiedź,
- oracle wynikał z logiki w wyrażeniu.

## Wnioski do AppSec

- Zawsze ustal baseline.
- Sprawdź faktyczny endpoint danych, nie tylko widoczny URL.
- Rozdziel dowód injection od dowodu impactu.
- Automatyzuj dopiero po ręcznym potwierdzeniu markera.
- Raportuj przyczynę źródłową jako niebezpieczne query construction, nie jako sam payload.
