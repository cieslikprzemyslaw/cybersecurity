# A04:2025 - Cryptographic Failures

## Co oznacza ta kategoria OWASP

Cryptographic Failures występują wtedy, gdy aplikacja nie chroni prawidłowo danych wrażliwych albo zaufanych tokenów.

Może to wynikać z braku kryptografii, słabej kryptografii, błędnego użycia, przestarzałych algorytmów, przewidywalnych wartości albo złej obsługi tokenów i kluczy.

W prostych słowach:

> A04 dotyczy sytuacji, w której dane wrażliwe albo zaufane tokeny nie są chronione wystarczająco mocno.

To nie oznacza, że AppSec engineer powinien wymyślać albo implementować własną kryptografię. W praktycznych review bezpieczniejsze podejście jest odwrotne:

> Nie buduj własnej kryptografii. Sprawdź, czy aplikacja poprawnie używa znanych, standardowych i sprawdzonych mechanizmów.

## Dlaczego to ma znaczenie

Aplikacje opierają kryptografię i projekt tokenów na wielu wrażliwych mechanizmach:

- przechowywanie haseł,
- ochrona sesji,
- cookies typu remember-me,
- linki resetowania hasła,
- linki weryfikacji email,
- tokeny API,
- JWT,
- podpisane cookies,
- szyfrowanie w tranzycie,
- szyfrowanie w spoczynku,
- ochrona sekretów i kluczy.

Słaby projekt kryptograficzny może prowadzić do:

- account takeover,
- łamania haseł,
- przejęcia sesji,
- ujawnienia danych wrażliwych,
- fałszowania tokenów,
- omijania granic zaufania,
- replay starych tokenów,
- ujawnienia sekretów.

## Kontekst praktycznego review

Dla tej kategorii ukończyłem dwa laby PortSwigger Web Security Academy dotyczące słabego projektu cookie remember-me:

1. **Offline password cracking**
2. **Brute-forcing a stay-logged-in cookie**

Oba laby używały słabego wzorca persistent-login cookie:

```text
stay-logged-in = Base64(username:MD5(password))
```

Problemem nie było tylko użycie Base64. Głębszy problem polegał na tym, że cookie zawierało przewidywalną wartość pochodzącą od hasła.

Cookie można było zdekodować, zrozumieć, odtworzyć i brute-force'ować.

## Kluczowe pojęcia, które przećwiczyłem

### Base64 to encoding, nie encryption

Base64 nie jest szyfrowaniem.

Nie używa sekretnego klucza. Jest zaprojektowane jako odwracalne kodowanie. Każdy, kto ma wartość, może ją zdekodować i odczytać oryginalne dane.

W labie zdekodowanie cookie ujawniało strukturę podobną do:

```text
username:md5(password)
```

To oznaczało, że cookie nie było zaszyfrowane ani chronione. Było tylko zakodowane.

### MD5 nie nadaje się do ochrony haseł

MD5 jest szybkim i przestarzałym algorytmem hashującym.

Nie nadaje się do ochrony haseł, ponieważ atakujący mogą bardzo szybko testować wiele kandydatów i porównywać wynikowy hash z hashem docelowym.

Dla haseł nowoczesne aplikacje powinny używać adaptacyjnego, solonego hashowania haseł, na przykład:

- Argon2,
- bcrypt,
- scrypt,
- PBKDF2 tam, gdzie ma to sens.

Jednak token remember-me w ogóle nie powinien bazować na hashu hasła.

### Projekt tokenu remember-me

Bezpieczniejszy token remember-me nie powinien wyglądać tak:

```text
Base64(username:MD5(password))
```

Bezpieczniejszy projekt powinien używać długiego, losowego tokenu o wysokiej entropii.

Serwer powinien przechowywać zahashowaną wersję tego tokenu i powiązać ją z:

- użytkownikiem,
- urządzeniem/sesją,
- czasem wygaśnięcia,
- stanem revocation,
- opcjonalnym zachowaniem rotacji.

Cookie powinno być chronione przez:

- `HttpOnly`,
- `Secure`,
- odpowiednie `SameSite`,
- czas wygaśnięcia.

## Typowe przykłady

Cryptographic Failures mogą obejmować:

- Base64 używane tak, jakby było szyfrowaniem,
- hasła przechowywane albo ujawniane ze słabymi hashami,
- MD5 albo SHA1 użyte do ochrony wartości związanych z hasłem,
- przewidywalne tokeny remember-me,
- wartości pochodzące od hasła przechowywane w cookies,
- przewidywalnie generowane tokeny resetu hasła,
- sekrety commitowane do kodu,
- słabe albo ponownie używane klucze,
- brak TLS albo słabą konfigurację TLS,
- dane wrażliwe wysyłane po HTTP,
- brak `Secure` na wrażliwych cookies,
- brak `HttpOnly` na cookies, których JavaScript nie powinien czytać,
- słabe generowanie losowości,
- własną kryptografię,
- tokeny, które nigdy nie wygasają albo nie mogą zostać odwołane.

## Typowe przyczyny źródłowe

Typowe przyczyny to:

- mylenie encoding z encryption,
- używanie starych przykładów albo legacy crypto patterns,
- używanie szybkich hashy dla wartości związanych z hasłem,
- przechowywanie danych pochodzących od hasła w cookies po stronie klienta,
- budowanie własnych formatów tokenów,
- brak walidacji tokenu po stronie serwera,
- brak bezpiecznych flag cookie,
- traktowanie przeglądarki jak bezpiecznego miejsca przechowywania,
- brak review projektu tokenu,
- brak expiry, rotation i revocation,
- słabe rozumienie różnicy między hashing, encryption i signing.

## Impact

Impact zależy od danych i tokenu, które miały być chronione.

W tych labach impactem było account takeover, ponieważ cookie remember-me można było użyć do dostępu do konta innego użytkownika.

Możliwe realne impacty:

- offline password cracking,
- fałszowanie persistent login token,
- account takeover,
- session hijacking,
- replay skradzionych tokenów,
- ujawnienie danych pochodzących od hasła,
- łatwiejsza eksploatacja w połączeniu z XSS,
- długotrwały nieautoryzowany dostęp, jeśli tokeny remember-me nie wygasają albo nie mogą zostać odwołane.

## Powiązane podatności, które już ćwiczyłem

Powiązane tematy z wcześniejszej nauki:

- [HTTP, Request/Response i podstawy Auth](../../fundamentals/01-http-request-response-auth.md)
- [Burp Suite, Proxy i podstawy Repeatera](../../fundamentals/02-burp-suite-proxy-repeater.md)
- [Authentication Bypass / Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Broken Access Control / IDOR](../../key-web-vulnerabilities/02-broken-access-control-idor/README.md)
- [Cross-Site Scripting](../../key-web-vulnerabilities/03-cross-site-scripting-xss/README.md)
- [CSRF + SameSite Cookies](../../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md)
- [Security Misconfiguration](../a02-security-misconfiguration/01-overview.md)

Te tematy są powiązane, ponieważ słaby projekt crypto/tokenów często staje się groźniejszy w połączeniu z problemami po stronie przeglądarki, takimi jak XSS, brak flag cookie albo słaba obsługa sesji.

## Mój takeaway jako Frontend Engineer

Najważniejsza lekcja frontendowa:

> Wszystko, co jest przechowywane w przeglądarce, trzeba traktować ostrożnie. Jeśli JavaScript może to odczytać, XSS prawdopodobnie też może to ukraść.

Frontend developer powinien rozumieć, że:

- Base64 nie chroni danych,
- wrażliwe tokeny nie powinny być przechowywane w storage czytelnym dla JavaScript bez mocnego powodu,
- cookies, które nie muszą być dostępne dla JavaScript, zwykle powinny mieć `HttpOnly`,
- tokeny remember-me nie powinny zawierać username, hashy haseł ani przewidywalnych struktur,
- projekt tokenów i cookies to decyzja backend/security, ale frontend developerzy często widzą symptomy w DevTools i Burp.

## Jak wygląda bezpieczniejszy stan

Bezpieczniejsza implementacja powinna zawierać:

- losowe tokeny remember-me o wysokiej entropii,
- walidację tokenu po stronie serwera,
- przechowywanie hasha tokenu po stronie serwera,
- expiry tokenu,
- revocation tokenu,
- rotację tokenu tam, gdzie ma to sens,
- brak hasha hasła w cookies po stronie klienta,
- brak przewidywalnego formatu tokenu,
- `HttpOnly` dla cookies, których JavaScript nie powinien czytać,
- `Secure` dla cookies wysyłanych tylko po HTTPS,
- odpowiednie `SameSite`,
- wymuszony TLS dla wrażliwych flow,
- nowoczesne hashowanie prawdziwych haseł,
- brak własnej kryptografii.

## Podsumowanie

Główna lekcja A04 z tych labów:

> Token, który wygląda na zakodowany albo skomplikowany, nie jest automatycznie bezpieczny. Jeśli format jest przewidywalny i bazuje na słabych albo pochodzących od hasła wartościach, może dać się zdekodować, odtworzyć, brute-force'ować albo ukraść.
