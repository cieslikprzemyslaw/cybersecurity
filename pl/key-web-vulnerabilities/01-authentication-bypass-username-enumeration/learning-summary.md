# Authentication Bypass i Username Enumeration - podsumowanie nauki

> Na podstawie legalnych labów treningowych TryHackMe i PortSwigger Web Security Academy.  
> Notatka nie zawiera flag, prawdziwych danych uwierzytelniających ani pełnych answer key.

## Overview

Ten temat pomógł mi zrozumieć, jak małe różnice w logowaniu mogą zdradzać informacje o użytkownikach.

Najważniejsze obszary:

- czym różni się authentication od authorization,
- czym jest authentication bypass,
- jak działa username enumeration,
- jak porównywać odpowiedzi w Burp Suite,
- dlaczego frontend nie jest granicą bezpieczeństwa,
- jak pisać developer-friendly remediation.

## Kluczowe pojęcia

Authentication odpowiada na pytanie:

> Kim jesteś?

Authorization odpowiada na pytanie:

> Co wolno Ci zrobić?

Username enumeration pojawia się wtedy, gdy aplikacja ujawnia, czy username albo email istnieje. Może to wynikać z różnych komunikatów, statusów, redirectów, długości odpowiedzi, timingu, cookies albo account lockout.

## Co testowałem

W labach porównywałem:

- requesty logowania,
- parametry `username` i `password`,
- odpowiedzi dla istniejących i nieistniejących użytkowników,
- komunikaty błędów,
- długość odpowiedzi,
- redirecty,
- cookies i sesje,
- zachowanie po wielu błędnych próbach.

## Najważniejsze lekcje

`200 OK` nie zawsze oznacza sukces biznesowy. Request mógł zostać obsłużony, ale logowanie mogło się nie udać.

Małe różnice mają znaczenie. Brakujący znak, inna długość HTML-a albo inny redirect mogą być wystarczającym sygnałem.

Frontend nie jest granicą bezpieczeństwa. Użytkownik może wysłać request przez Burpa, Postmana, curl albo skrypt, więc backend musi egzekwować security decisions.

Narzędzia pomagają, ale analiza jest ważniejsza. Burp Intruder albo ffuf mogą wysłać wiele requestów, ale to człowiek musi zrozumieć, co oznacza różnica w odpowiedzi.

## Impact

Username enumeration może pomóc atakującemu:

- zidentyfikować prawdziwe konta,
- skupić password guessing na realnych użytkownikach,
- przygotować phishing,
- ułatwić credential stuffing,
- nadużyć account lockout.

Authentication bypass może być znacznie poważniejszy, jeśli pozwala wejść do konta albo funkcji chronionej bez poprawnego uwierzytelnienia.

## Remediacja

Bezpieczniejsze flow logowania powinno:

- używać generycznego komunikatu błędu,
- utrzymywać spójne status code,
- unikać różnic w redirectach i cookies,
- ograniczać oczywiste różnice długości odpowiedzi,
- projektować account lockout tak, aby nie potwierdzał istnienia konta,
- stosować rate limiting,
- logować podejrzane próby,
- rozważyć MFA dla wrażliwych kont,
- walidować sesję po stronie serwera na każdym chronionym endpointcie.

## Główna lekcja

Testowanie authentication polega na analizie zachowania aplikacji, nie tylko na patrzeniu na komunikaty błędów.

Login form może zdradzać informacje przez tekst, długość response, redirect, cookies, timing albo lockout. Bezpieczna implementacja powinna minimalizować te różnice i egzekwować logikę po stronie backendu.
