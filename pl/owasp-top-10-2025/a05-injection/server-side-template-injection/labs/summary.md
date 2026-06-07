# SSTI Practical Work Summary

## Ukończone cwiczenia

| Platforma | Engine | Runtime | Glowny dowod | Pokazany glebszy impact |
|---|---|---|---|---|
| TryHackMe | Smarty | PHP | Template modifier/expression zmienil output | Funkcja PHP w labie doprowadzila do command execution |
| TryHackMe | Pug | Node.js | Wyrazenie JavaScript zwrocilo wynik | Node process API doprowadzilo do command execution |
| TryHackMe | Jinja2 | Python | Wyrazenie szablonu zwrocilo wynik | Python object traversal doprowadzil do `subprocess` |
| PortSwigger | ERB | Ruby | Wyrazenie arytmetyczne zwrocilo `49` | Ruby `File.delete()` usunelo wskazany plik |

## Wspolny model mentalny

```text
controlled input
  -> template engine evaluates syntax
  -> runtime objects or APIs may become reachable
  -> impact depends on context, configuration, and privileges
```

## Najbardziej uzyteczna lekcja cross-language

Jezyk i API sie roznia, ale rozumowanie AppSec zostaje podobne:

1. Znajdz kontrolowany input.
2. Znajdz miejsce renderowania.
3. Oddziel reflection od evaluation.
4. Zidentyfikuj engine na podstawie dowodow.
5. Udowodnij jedna mozliwosc naraz.
6. Nie nazywaj evaluation pelnym RCE bez dowodow.
7. Oddziel root cause od impactu.
8. Napraw granice data/template-source.

## Dlaczego jeden lab PortSwigger wystarczy teraz

Lab PortSwigger dodal pelny black-box reasoning flow i czwarty engine/runtime po trzech praktycznych cwiczeniach TryHackMe.

Aktualne pokrycie obejmuje:

- wiele skladni szablonow,
- kilka jezykow serwerowych,
- reflection versus evaluation,
- fingerprinting concepts,
- bezposrednie runtime APIs,
- roznice command/process API,
- root cause,
- remediation,
- regression testing.

Kolejne laby dodalyby glebokosc, ale nie sa wymagane przed kontynuacja obecnego roadmap A05.
