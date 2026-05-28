# A04:2025 - Cryptographic Failures checklist

## Cel

Używaj tej checklisty podczas review aplikacji, API, cookies, tokenów, flow uwierzytelniania i frontend storage pod kątem Cryptographic Failures.

Celem jest wykrycie słabej ochrony danych wrażliwych i słabego projektu tokenów.

## Główne pytanie

Zapytaj:

> Czy dane wrażliwe albo zaufany token są chronione mocnymi, standardowymi, nieprzewidywalnymi i poprawnie skonfigurowanymi mechanizmami?

## 1. Encoding vs Encryption

### Pytania testowe

- Czy aplikacja używa Base64 do ukrywania danych wrażliwych?
- Czy token albo cookie da się zdekodować do czytelnych wartości?
- Czy dekodowanie ujawnia username, ID, role, hashe, emaile, sekrety albo wewnętrzną strukturę?
- Czy encoding jest mylony z encryption?

### Red flags

```text
Base64(username:password)
Base64(username:hash)
Base64(userId:role)
Base64(email:token)
```

### Bezpieczna notatka

Base64 nie jest szyfrowaniem.

Jeśli wartość można zdekodować bez sekretnego klucza, nie powinna być traktowana jako chroniona.

## 2. Review cookie remember-me

### Pytania testowe

- Co jest przechowywane w cookie remember-me?
- Czy cookie dekoduje się do czytelnych danych?
- Czy zawiera username?
- Czy zawiera hash hasła?
- Czy zawiera przewidywalną strukturę?
- Czy atakujący może wygenerować token?
- Czy token jest losowy i ma wysoką entropię?
- Czy token jest walidowany po stronie serwera?
- Czy token wygasa?
- Czy token można odwołać?
- Czy token jest rotowany po użyciu albo po wrażliwych akcjach?

### Zły wzorzec

```text
Base64(username:MD5(password))
```

### Lepszy wzorzec

```text
random high-entropy token
    -> stored as a hash server-side
    -> linked to user/device/session
    -> expires
    -> revocable
    -> protected with secure cookie flags
```

## 3. Review hashowania

### Pytania testowe

- Czy MD5 jest używany dla wartości związanych z hasłem?
- Czy SHA1 jest używany dla wartości związanych z hasłem?
- Czy szybkie hashe są używane tam, gdzie potrzebne jest wolne hashowanie haseł?
- Czy hasła są solone?
- Czy użyto adaptacyjnej funkcji hashowania haseł?
- Czy hash jest ujawniany klientowi?
- Czy wartość pochodząca od hasła jest przechowywana w cookie albo tokenie?

### Red flags

```text
MD5(password)
SHA1(password)
unsalted password hash
password hash in cookie
password hash in URL
password hash in localStorage
```

### Bezpieczniejsze przechowywanie haseł

Dla przechowywania haseł używaj standardowych algorytmów hashowania haseł, takich jak:

- Argon2,
- bcrypt,
- scrypt,
- PBKDF2 tam, gdzie ma to sens.

Nie używaj hashy haseł jako tokenów remember-me.

## 4. Review flag cookie

### Pytania testowe

- Czy cookie musi być czytelne dla JavaScript?
- Jeśli nie, czy ustawiono `HttpOnly`?
- Czy ustawiono `Secure` dla transmisji tylko po HTTPS?
- Czy `SameSite` pasuje do legalnego flow aplikacji?
- Czy cookie ma sensowny czas wygaśnięcia?
- Czy cookie zawiera dane wrażliwe albo przewidywalne?

### Ważne notatki

`HttpOnly` nie naprawia XSS.

Zapobiega jednak odczytowi cookie przez JavaScript przez:

```js
document.cookie
```

Może to zmniejszyć impact XSS wobec cookies sesyjnych albo remember-me.

## 5. Losowość i przewidywalność tokenów

### Pytania testowe

- Czy token wygląda losowo?
- Czy potrafię przewidzieć format tokenu?
- Czy mogę odtworzyć token, jeśli znam username?
- Czy mogę generować tokeny z użyciem wordlisty?
- Czy token zawiera timestampy, ID albo hashe?
- Czy token jest generowany z kryptograficznie bezpiecznego źródła losowości?
- Czy token jest wystarczająco długi?

### Red flags

- token zawiera username,
- token zawiera user ID,
- token zawiera rolę,
- token zawiera hash hasła,
- token ma jasny delimiter typu `username:hash`,
- token można zdekodować z Base64 do znaczących danych,
- token można brute-force'ować offline.

## 6. Review frontend storage

### Pytania testowe

- Czy sekrety są przechowywane w frontend JavaScript?
- Czy access tokens są przechowywane w localStorage?
- Czy long-lived tokens są czytelne dla JavaScript?
- Czy refresh tokens są wystawione na JavaScript?
- Czy tokeny trafiają do URL?
- Czy wartości wrażliwe trafiają do logów przeglądarki?
- Czy tokeny znajdują się w source maps albo bundlach?

### Red flags

```text
localStorage.setItem('token', ...)
window.__CONFIG__.secret
API key in JS bundle
long-lived token in localStorage
password hash in browser storage
```

## 7. Pytania testowe w Burp

Podczas review cookies albo tokenów w Burp:

- Co się stanie, jeśli usunę session cookie i zostawię tylko remember-me?
- Co się stanie, jeśli zdekoduję cookie z Base64?
- Czy zdekodowana wartość ma strukturę?
- Czy mogę zmienić username i odbudować cookie?
- Czy mogę użyć Intrudera do generowania candidate cookies?
- Czy udane i nieudane próby tokenu mają różne długości odpowiedzi?
- Czy odpowiedź ujawnia uwierzytelniony username?

## 8. Wzorzec review w Burp Intruder

Jeśli cookie ma taki wzorzec:

```text
Base64(username:MD5(password))
```

to Burp Intruder może zostać użyty do testowania haseł-kandydatów przez przetworzenie każdego payloadu:

```text
candidate password
    -> MD5
    -> add prefix: username:
    -> Base64 encode
```

W bezpiecznym projekcie nie powinno to być możliwe.

## 9. Remediation checklist

Dla słabego projektu tokenu remember-me:

- Usuń wartości pochodzące od hasła z cookies.
- Przestań używać tokenów w stylu `Base64(username:hash)`.
- Generuj losowe tokeny o wysokiej entropii.
- Przechowuj po stronie serwera tylko zahashowane tokeny.
- Powiąż tokeny z user/session/device tam, gdzie ma to sens.
- Dodaj expiry.
- Dodaj revocation.
- Rotuj tokeny po wrażliwych zdarzeniach.
- Użyj `HttpOnly`, `Secure` i odpowiedniego `SameSite`.
- Unieważnij stare słabe tokeny.
- Przejrzyj logi pod kątem możliwego nadużycia.

Dla słabego hashowania:

- Nie używaj MD5 ani SHA1 dla haseł.
- Użyj Argon2, bcrypt, scrypt albo odpowiedniego PBKDF2.
- Używaj soli i work factors.
- Nie ujawniaj hashy haseł klientowi.

Dla ryzyka XSS + cookie theft:

- Napraw XSS.
- Ustaw `HttpOnly` tam, gdzie JavaScript nie potrzebuje dostępu.
- Użyj CSP jako defence-in-depth.
- Unikaj przechowywania wrażliwych long-lived tokens w miejscach czytelnych dla JavaScript.

## 10. Developer red flags

Uważaj, gdy widzisz:

- wartości Base64 w cookies,
- tokeny remember-me z czytelną strukturą,
- MD5 albo SHA1 w flow uwierzytelniania,
- wartości cookie zawierające username i hashe,
- long-lived tokens bez expiry,
- tokeny bez revocation,
- brak `HttpOnly`,
- wartości tokenów w URL,
- sekrety w frontend bundles,
- custom crypto helpers,
- komentarze sugerujące "encoded for security".

## Wniosek frontendowy

Frontend developerzy zwykle nie projektują hashowania haseł ani przechowywania tokenów remember-me, ale często widzą dowody:

- cookies w DevTools,
- localStorage/sessionStorage,
- wartości tokenów w requestach,
- brakujące flagi cookie,
- wartości wyglądające jak Base64,
- impact XSS na tokeny czytelne dla przeglądarki.

Jeśli token jest czytelny dla JavaScript i ma przewidywalną strukturę, zasługuje na review.
