# TryHackMe A09 Practice

## Źródło

TryHackMe - OWASP Top 10 2025: IAAA Failures, sekcja A09.

## Zakres

Ukończyłem materiał A09 po wcześniejszym zaliczeniu części A01 i A07 tego pokoju. Korzystałem z pokoju razem ze stroną OWASP A09 oraz obiema cheat sheetami OWASP dotyczącymi logowania.

Kluczowa nauka nie polegała na założeniu, że jeden konkretny komercyjny stack do monitoringu jest wymagany. Aplikacja najpierw musi emitować użyteczne security eventy, a dopiero potem szerszy system powinien je zbierać, monitorować, korelować, alertować i wspierać reakcję.

## Główne obserwacje

- Brak logów utrudnia albo uniemożliwia odtworzenie ataku.
- Logi bez monitoringu i alertingu mogą pozostać niezauważone podczas aktywnego ataku.
- Trzeba brać pod uwagę zarówno ważne sukcesy, jak i porażki.
- Logowanie danych wrażliwych tworzy nowy risk dla confidentiality.
- Niezaufane wartości mogą zaatakować sam pipeline logowania.
- Lokalne logi mogą być niedostępne, usunięte albo trudne do korelacji.
- Zbyt dużo false positives może ukryć ważne alerty.
- Detection bez aktualnego playbooka tworzy lukę w response.

## Standard dowodu

Dla A09 użyteczny dowód powinien obejmować:

```text
oczekiwane security action
    -> wysłany event
    -> zebrany event
    -> pasująca reguła detekcji
    -> dostarczony alert
    -> udokumentowana reakcja
```

Nie twierdzę, że atak został wykryty tylko dlatego, że istniał jeden event.

## Wynik

Sekcja TryHackMe dała kontekst dla kategorii. Praktyczne zrozumienie zostało dopięte przez plan authentication, review log injection oraz ćwiczenia klasyfikacji danych wrażliwych opisane w tym folderze.
