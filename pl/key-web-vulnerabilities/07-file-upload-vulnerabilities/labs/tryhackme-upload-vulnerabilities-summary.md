# TryHackMe Upload Vulnerabilities - podsumowanie nauki

## Zakres

Ten room wprowadzał typowe ryzyka uploadu plików i techniki filtrowania.

VM nie działała stabilnie podczas nauki, więc room został użyty głównie jako materiał do czytania, a praktyka została przeniesiona do labów PortSwigger.

## Kluczowe pojęcia

- ryzyko nadpisania pliku
- RCE przez uploadowane pliki
- filtrowanie po stronie klienta
- filtrowanie rozszerzeń po stronie serwera
- walidacja MIME / `Content-Type`
- magic bytes
- metodologia black-box

## Bypassy filtrowania po stronie klienta

- wyłączenie JavaScriptu
- modyfikacja odpowiedzi z kodem strony w Burp
- modyfikacja requestu uploadu w Burp
- wysłanie requestu bezpośrednio do endpointu uploadu

## Główna lekcja

Zaakceptowany upload nie wystarczy. Trzeba sprawdzić finalną nazwę pliku, miejsce zapisu, `Content-Type` odpowiedzi oraz to, czy plik jest wykonywany, czy serwowany statycznie.
