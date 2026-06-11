# A08: Software or Data Integrity Failures

Ten katalog zawiera moje praktyczne notatki do OWASP Top 10 2025 - A08: Software or Data Integrity Failures.

## Status

**PASS — Frontend Developer przechodzący do AppSec**

Ukończona praca:

- TryHackMe: OWASP Top 10 2025 - Insecure Data Handling, Task 4: A08,
- PortSwigger: Modifying serialized objects,
- przegląd integralności, autentyczności, checksum, podpisów cyfrowych, szyfrowania, skryptów zewnętrznych, Subresource Integrity, stanu kontrolowanego przez klienta i weryfikacji artefaktów,
- porównanie A03 Software Supply Chain Failures z A08 Software or Data Integrity Failures.

Status nie oznacza eksperckiej wiedzy o kryptografii, bezpieczeństwie CI/CD, gadget chains ani exploit development. Oznacza, że potrafię wskazać podstawowe granice zaufania związane z integralnością, rozpoznać niebezpieczną deserializację, oddzielić fakty od założeń, zaproponować praktyczne zabezpieczenia i napisać użyteczne testy regresyjne.

## Główny model mentalny

```text
artefakt oprogramowania lub dane
    -> granica zaufania
    -> weryfikacja integralności i autentyczności
    -> zaufane przetwarzanie
```

Najważniejsze pytania kontrolne:

1. Jaki software, konfiguracja, obiekt, cookie albo dane są traktowane jako zaufane?
2. Skąd pochodzą?
3. Kto może je zmienić?
4. Jak weryfikowana jest ich integralność?
5. Jak potwierdzane jest źródło?
6. Czy weryfikacja następuje przed użyciem lub deserializacją?
7. Czy stary, ale wcześniej poprawny artefakt można odtworzyć ponownie?
8. Co dzieje się po nieudanej weryfikacji?
9. Jaki dowód potwierdza, że zmienione dane zostały zaakceptowane?
10. Co powinno być źródłem prawdy w bezpiecznym projekcie?

## Perspektywa frontendowa

Przeglądarka nie jest zaufanym źródłem autorytetu.

- Base64 jest kodowaniem, a nie zabezpieczeniem.
- Hidden fields, cookies, `localStorage` i `sessionStorage` są kontrolowane przez użytkownika.
- Role, ceny, feature flags i uprawnienia wygenerowane przez frontend nie mogą być autorytatywne.
- Załadowanie skryptu zewnętrznego oznacza zaufanie kodowi wykonywanemu w kontekście aplikacji.
- Przypięcie wersji wspiera powtarzalność, ale samo nie dowodzi integralności artefaktu.
- SRI może zweryfikować statyczny zasób przeglądarkowy względem oczekiwanego hasha, ale nie jest uniwersalnym zabezpieczeniem łańcucha dostaw oprogramowania.

Podatność istnieje wtedy, gdy zaufany komponent akceptuje zmienione lub niezaufane dane jako autorytatywne bez wystarczającej weryfikacji. Sama obecność danych kontrolowanych przez klienta nie jest automatycznie podatnością.

## Ukończone wzorce praktyczne

### Świadomość ryzyka Python pickle

Zadanie TryHackMe pokazało, dlaczego dane Python `pickle` pochodzące od niezaufanego użytkownika są niebezpieczne. Rekonstrukcja obiektu może uruchamiać zachowanie podczas deserializacji. Mój wygenerowany payload został odrzucony błędem `UnpicklingError`, dlatego nie twierdzę, że samodzielnie udowodniłem odczyt pliku na serwerze. Zachowana lekcja dotyczy granicy zaufania: niezaufany serializowany input nie powinien trafiać do natywnej deserializacji obiektów.

### Obiekt sesji PHP kontrolowany przez klienta

Lab PortSwigger przechowywał serializowany obiekt PHP `User` w cookie zakodowanym przez URL i Base64:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

Zmiana wyłącznie:

```text
b:0
```

na:

```text
b:1
```

spowodowała:

- pojawienie się linku `/admin`,
- zwrócenie panelu administratora,
- możliwość usunięcia innego użytkownika.

Prawdziwym problemem nie było jedynie to, że cookie dało się edytować. Backend deserializował stan kontrolowany przez klienta i ufał zmienionej właściwości `admin` jako stanowi autoryzacji.

## Zacznij tutaj

- [Overview](01-overview.md)
- [Integralność i autentyczność](02-integrity-and-authenticity.md)
- [A03 kontra A08](03-a03-vs-a08.md)
- [Niebezpieczna deserializacja](04-insecure-deserialization.md)
- [Integralność software i artefaktów](05-software-and-artifact-integrity.md)
- [Checklista](06-checklist.md)
- [Testy regresyjne](07-regression-tests.md)
- [Learning notes](08-learning-notes.md)
- [Laby i praktyka](labs-and-practice/README.md)
- [Przykładowy security finding](security-findings/01-example-finding-insecure-deserialization.md)

## Linki do nauki

- [OWASP A08:2025 - Software or Data Integrity Failures](https://owasp.org/Top10/2025/A08_2025-Software_or_Data_Integrity_Failures/)
- [TryHackMe - OWASP Top 10 2025: Insecure Data Handling](https://tryhackme.com/room/owasptopten2025three)
- [PortSwigger - Insecure Deserialization](https://portswigger.net/web-security/deserialization)
- [PortSwigger Lab - Modifying serialized objects](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects)
