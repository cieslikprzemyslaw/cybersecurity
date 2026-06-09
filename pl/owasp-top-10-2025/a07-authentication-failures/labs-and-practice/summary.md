# Podsumowanie Praktyki A07

## Ukończony zakres

- Rozróżnienia IAAA: identification, authentication, authorization, accountability.
- Password reset jako alternatywny mechanizm authentication.
- Powiązanie reset tokenu z właściwym kontem.
- Dowody potrzebne do potwierdzenia account takeover.
- Stan MFA-pending a fully authenticated session.
- Server-side enforcement MFA na chronionych zasobach.
- Frontend controls a backend security boundaries.
- Session rotation i logout invalidation na poziomie awareness/review.
- Enumeration i rate limiting na poziomie awareness/review.

## Najważniejsze lekcje praktyczne

1. Wartości tożsamości przesyłane z przeglądarki są deklaracją, nie zaufanym dowodem.
2. Recovery evidence musi autoryzować jedną tożsamość i jedną akcję.
3. Wyświetlenie kroku bezpieczeństwa nie oznacza, że backend go egzekwuje.
4. Redirect lub nietypowy response jest dowodem zachowania, ale nie zawsze dowodem finalnego impactu.
5. Account takeover deklarujemy dopiero po kontrolowanym potwierdzeniu dostępu.

## Ocena końcowa

**PASS** dla praktycznej podstawy oczekiwanej od Frontend Developera przechodzącego do AppSec.
