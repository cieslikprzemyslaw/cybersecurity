# Evidence, Impact and Severity

## Dowody

Dobry raport Prompt Injection powinien oddzielać:

- co model wygenerował,
- co aplikacja rzeczywiście wykonała,
- jakie dane były dostępne,
- jakie uprawnienia miał użytkownik,
- czy nastąpiła zmiana stanu,
- czy impact jest powtarzalny.

## Słabe dowody

- "Model powiedział, że ominął zabezpieczenia".
- "Model wygenerował komendę".
- "Model obiecał wysłać email".

To są sygnały do dalszej weryfikacji, nie potwierdzony impact.

## Silne dowody

- Nieautoryzowany rekord został pobrany.
- Tool call został wykonany z niedozwolonym argumentem.
- Dane innego użytkownika pojawiły się w odpowiedzi.
- Agent wykonał akcję, której użytkownik nie powinien móc wykonać.
- Retrieved content zmienił decyzję narzędzia albo zakres danych.

## Severity

Severity zależy od:

- wrażliwości danych,
- możliwości tool execution,
- uprawnień agenta,
- wpływu na wielu użytkowników,
- trwałości skutku,
- możliwości wykrycia i cofnięcia działania.
