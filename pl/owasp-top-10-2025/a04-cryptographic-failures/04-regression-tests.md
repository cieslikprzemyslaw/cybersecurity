# A04:2025 - Cryptographic Failures: testy regresji

## Cel

Testy regresji dla Cryptographic Failures powinny potwierdzać, że dane wrażliwe i zaufane tokeny nie są chronione słabymi, przewidywalnymi albo odwracalnymi mechanizmami.

Dla tych labów głównym tematem jest projekt cookie remember-me.

## Obszar 1: cookie remember-me nie może ujawniać przewidywalnej struktury

### Test negatywny: cookie nie dekoduje się do username i hasha

**Given** użytkownik loguje się z włączonym "remember me"  
**When** aplikacja wydaje persistent login cookie  
**Then** dekodowanie cookie z Base64 nie powinno ujawniać:

```text
username
password hash
MD5 hash
email
role
user ID
username:hash
```

### Oczekiwany rezultat

Wartość cookie powinna wyglądać jak losowy, opaque token.

Nie powinna mieć znaczącej struktury, którą atakujący może odtworzyć.

## Obszar 2: wygenerowane cookie Base64(username:MD5(password)) musi zawieść

### Test negatywny: przewidywalne candidate cookie jest odrzucane

**Given** atakujący zna stary słaby format:

```text
Base64(username:MD5(password))
```

**When** atakujący wysyła wygenerowane cookie, na przykład:

```http
Cookie: stay-logged-in=<base64-username-md5-candidate>
```

**Then** aplikacja nie może uwierzytelnić użytkownika.

### Oczekiwany rezultat

Odpowiedź nie powinna zawierać:

```html
<p>Your username is: carlos</p>
```

Request powinien skończyć się login prompt, invalid token response albo stanem nieuwierzytelnionym.

## Obszar 3: flagi cookie

### Test pozytywny: cookie remember-me używa bezpiecznych flag

**Given** cookie remember-me jest wydawane  
**Then** cookie powinno zawierać:

```text
HttpOnly
Secure
SameSite=<appropriate-value>
```

### Notatki

- `HttpOnly` zapobiega odczytowi cookie przez JavaScript przez `document.cookie`.
- `Secure` zapewnia, że cookie jest wysyłane tylko po HTTPS.
- `SameSite` powinno pasować do legalnych potrzeb cross-site aplikacji.

## Obszar 4: XSS nie może odczytać wrażliwego cookie

### Test negatywny: JavaScript nie może odczytać cookie remember-me

**Given** strona wykonuje JavaScript  
**When** skrypt odczytuje:

```js
document.cookie
```

**Then** cookie remember-me nie powinno pojawić się w wyniku, jeśli JavaScript nie potrzebuje do niego dostępu.

To nie zastępuje naprawy XSS, ale zmniejsza impact kradzieży cookie.

## Obszar 5: token jest losowy i walidowany po stronie serwera

### Test pozytywny: token jest opaque

**Given** dwóch użytkowników loguje się z włączonym remember-me  
**Then** ich cookies remember-me nie powinny ujawniać tożsamości użytkownika ani wartości pochodzących od hasła.

### Test pozytywny: token jest bezpiecznie przechowywany po stronie serwera

Serwer powinien walidować token wobec storage po stronie serwera.

Zalecane zachowanie:

- przechowywać hash tokenu po stronie serwera,
- powiązać token z user/session/device,
- śledzić expiry,
- umożliwiać revocation,
- rotować token tam, gdzie ma to sens.

## Obszar 6: expiry i revocation tokenu

### Test pozytywny: wygasły token jest odrzucany

**Given** token remember-me wygasł  
**When** zostaje przesłany  
**Then** aplikacja powinna go odrzucić.

### Test pozytywny: logout odwołuje token

**Given** użytkownik się wylogowuje  
**When** stary token remember-me zostaje użyty ponownie  
**Then** token nie powinien uwierzytelnić użytkownika.

### Test pozytywny: zmiana hasła odwołuje stare tokeny

**Given** użytkownik zmienia hasło  
**When** stary token remember-me zostaje użyty  
**Then** stary token powinien zostać odrzucony.

## Obszar 7: słabe hashe nie są używane do ochrony haseł

### Static albo code review

Szukaj użycia związanego z hasłami:

```text
md5
sha1
crypto.createHash('md5')
crypto.createHash('sha1')
```

### Oczekiwany rezultat

Password storage powinien używać mocnej adaptacyjnej funkcji hashowania haseł, takiej jak:

- Argon2,
- bcrypt,
- scrypt,
- PBKDF2 tam, gdzie ma to sens.

Hashe haseł nie powinny być wysyłane do klienta ani przechowywane w cookies.

## Obszar 8: tokeny generowane stylem Intruder nie działają

### Test negatywny

Utwórz mały zestaw wygenerowanych cookies w starym słabym formacie:

```text
Base64(carlos:MD5(password1))
Base64(carlos:MD5(password2))
Base64(carlos:MD5(password3))
```

Wyślij każde jako:

```http
GET /my-account?id=carlos
Cookie: stay-logged-in=<generated-cookie>
```

### Oczekiwany rezultat

Wszystkie wygenerowane cookies w słabym formacie powinny zawieść.

Żadna odpowiedź nie powinna uwierzytelnić requestera jako Carlos.

## Obszar 9: brak danych wrażliwych w browser storage

### Weryfikacja manualna

Sprawdź:

- cookies,
- localStorage,
- sessionStorage,
- JavaScript globals,
- source maps,
- browser console output.

### Oczekiwany rezultat

Te miejsca nie powinny zawierać:

- hashy haseł,
- raw secrets,
- long-lived sensitive tokens czytelnych dla JavaScript,
- przewidywalnych authentication tokens.

## Co powinno failować po poprawce

Po remediacji powinno zawieść:

```text
Base64 decoding a remember-me cookie into username:hash
Using Base64(username:MD5(password)) to authenticate
Reading the remember-me cookie through document.cookie
Using an old remember-me token after logout
Using an old remember-me token after password change
```

## Co powinno nadal działać

Te rzeczy powinny nadal działać:

```text
Normal login
Legitimate remember-me login with a valid random token
Logout
Password change
Token expiry
Token revocation
```

## Acceptance criteria

Remediation można zaakceptować, gdy:

- tokeny remember-me są losowe i opaque,
- cookies nie zawierają wartości pochodzących od hasła,
- Base64 decoding nie ujawnia wrażliwej struktury,
- wygenerowane cookies `username:MD5(password)` zawodzą,
- cookies używają `HttpOnly`, `Secure` i odpowiedniego `SameSite`,
- tokeny wygasają,
- tokeny można odwołać,
- zmiana hasła unieważnia stare persistent login tokens,
- hashe haseł nie są ujawnione przeglądarce.

## Podsumowanie

Bezpieczna poprawka nie powinna tylko zmieniać encoding.

Projekt musi zmienić się z:

```text
predictable client-side token based on username and password hash
```

na:

```text
random server-side validated token with expiry, revocation and secure cookie flags
```
