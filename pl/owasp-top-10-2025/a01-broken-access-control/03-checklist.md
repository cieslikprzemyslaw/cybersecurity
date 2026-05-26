# A01 Checklista: Broken Access Control

Używaj tej checklisty podczas code review, API review, testów manualnych albo review AppSec.

Główne pytanie brzmi:

> Czy użytkownik może wykonać akcję albo uzyskać dostęp do zasobu poza swoimi zamierzonymi uprawnieniami?

## 1. Zrozum funkcję

Przed testowaniem ustal:

- Jaka akcja jest wykonywana?
- Czy akcja zmienia stan?
- Czy akcja jest read-only, ale wrażliwa?
- Kto powinien móc ją wykonać?
- Czy reguła opiera się na roli, ownership, organizacji, tenant, statusie lub stanie workflow?
- Który endpoint backendowy faktycznie wykonuje akcję?
- Czy funkcja jest częścią wieloetapowego workflow?

Przykłady wrażliwych akcji:

- zmiany ról,
- usunięcie konta,
- zarządzanie użytkownikami,
- podgląd faktur lub dokumentów,
- zmiana emaila lub hasła,
- pobieranie plików,
- publikowanie treści,
- zatwierdzanie/odrzucanie zgłoszeń,
- zmiany płatności lub billingowe,
- akcje administracyjnej moderacji.

## 2. Authentication vs authorization

Sprawdź, czy zespół nie miesza authentication i authorization.

Pytania authentication:

- Czy użytkownik jest zalogowany?
- Czy sesja jest ważna?
- Czy token jest ważny?

Pytania authorization:

- Czy ten zalogowany użytkownik może wykonać tę akcję?
- Czy ten użytkownik może uzyskać dostęp do tego konkretnego zasobu?
- Czy ten użytkownik może zmodyfikować ten konkretny obiekt?
- Czy ten użytkownik może ukończyć ten krok workflow?

Użytkownik może być poprawnie uwierzytelniony, ale nadal nieautoryzowany.

## 3. Pytania do code review

Zapytaj:

- Gdzie wykonywane jest sprawdzenie autoryzacji?
- Czy jest egzekwowane na backendzie?
- Czy jest egzekwowane na każdym wrażliwym endpoincie?
- Czy finalny endpoint zmieniający stan jest chroniony?
- Czy ownership checks są wykonywane server-side?
- Czy role checks są wykonywane server-side?
- Czy logika jest deny-by-default?
- Czy istnieje centralny guard, middleware, policy lub check na poziomie service?
- Czy checks są duplikowane niespójnie między controllerami?
- Czy użytkownik może zmienić `userId`, `username`, `role`, `tenantId`, `accountId`, `orgId` lub `confirmed` w requeście?
- Czy backend ufa czemukolwiek pochodzącemu od klienta przy decyzjach autoryzacyjnych?
- Czy authorization checks są pokryte testami automatycznymi?

## 4. Pytania testowe

Dla każdego wrażliwego requestu przetestuj:

- Co dzieje się jako użytkownik nieuwierzytelniony?
- Co dzieje się jako normalny uwierzytelniony użytkownik?
- Co dzieje się jako właściciel zasobu?
- Co dzieje się jako inny użytkownik?
- Co dzieje się jako administrator?
- Co dzieje się po zmianie ID?
- Co dzieje się po zmianie metody HTTP?
- Co dzieje się, gdy request jest wysłany bezpośrednio, bez UI?
- Co dzieje się, gdy wysłany jest tylko finalny krok potwierdzenia?
- Co dzieje się po replay requestu?
- Co dzieje się po modyfikacji parametrów wyglądających na opcjonalne?
- Czy aplikacja zwraca `403 Forbidden` dla uwierzytelnionych, ale nieautoryzowanych użytkowników?
- Czy aplikacja pozostawia stan bez zmian po odrzuceniu requestu?

## 5. Checklista wieloetapowego workflow

Dla workflow takich jak zmiany ról, approvals, usuwanie konta lub publikowanie:

- Zidentyfikuj każdy request w workflow.
- Zidentyfikuj request, który faktycznie zmienia stan.
- Przetestuj każdy krok z użytkownikiem o niższych uprawnieniach.
- Przetestuj finalny krok potwierdzenia bezpośrednio.
- Sprawdź, czy backend polega na `confirmed=true` lub podobnych wartościach kontrolowanych przez klienta.
- Sprawdź, czy wcześniejsza autoryzacja jest zakładana zamiast ponownie sprawdzana.
- Potwierdź, że nieautoryzowane requesty finalnego kroku kończą się niepowodzeniem.
- Potwierdź, że underlying state się nie zmienia.
- Potwierdź, że podejrzane próby są logowane tam, gdzie to właściwe.

## 6. Frontend-specific checks

Sprawdzenia frontendowe są użyteczne, ale nie są wystarczającymi kontrolami bezpieczeństwa.

Review:

- Czy linki admina są ukryte przed normalnymi użytkownikami?
- Czy chronione route'y są guardowane dla UX?
- Czy frontend poprawnie obsługuje `401` i `403`?
- Czy UI przypadkowo ujawnia ścieżki endpointów admina?
- Czy UI polega na client-side wartościach ról, które da się zmodyfikować?
- Czy UI zapobiega przypadkowym akcjom?

Ale potwierdź też:

- Backend egzekwuje ten sam model uprawnień.
- Endpointy API odrzucają nieautoryzowane bezpośrednie requesty.
- Wrażliwe akcje nie zależą tylko od ukrytych przycisków, disabled controls lub route guardów.

## 7. Developer red flags

Red flags:

- "Użytkownik nie widzi przycisku, więc nie może tego zrobić."
- "Tylko admini znają ten URL."
- "Pierwszy krok sprawdza admina, więc confirmation step jest ok."
- "Frontend wysyła rolę, więc backend jej ufa."
- "Request pochodzi z naszego UI, więc jest bezpieczny."
- "Endpoint nie jest nigdzie podlinkowany."
- "Sprawdzamy uprawnienia przy ładowaniu strony, nie przy akcji."
- "To tylko endpoint wewnętrzny."
- "User ID pochodzi z hidden field."
- "Rola jest disabled w UI, więc użytkownik nie może jej zmienić."

## 8. Checklista remediacji

Bezpieczniejsza implementacja powinna:

- egzekwować autoryzację server-side,
- używać deny-by-default dla wrażliwych akcji,
- sprawdzać aktualnego uwierzytelnionego użytkownika, a nie tylko parametry requestu,
- sprawdzać rolę i ownership tam, gdzie to właściwe,
- chronić każdy endpoint wykonujący wrażliwą akcję,
- chronić finalne kroki potwierdzenia w wieloetapowych workflow,
- centralizować logikę autoryzacji tam, gdzie to możliwe,
- nie ufać wartościom kontrolowanym przez klienta przy decyzjach permission,
- zwracać `403 Forbidden`, gdy uwierzytelniony użytkownik nie ma wymaganych uprawnień,
- upewnić się, że stan aplikacji nie zmienia się po nieautoryzowanych requestach,
- logować podejrzane nieautoryzowane próby dla high-risk actions,
- dodać testy regresji dla bezpośrednich requestów backendowych.

## 9. Notatki bezpiecznej implementacji

Dobre wzorce:

- współdzielony authorization middleware lub guards,
- policy-based checks, np. `canUpdateUser(currentUser, targetUser)`,
- object-level authorization checks,
- tenant/organisation scoping w query,
- server-side role checks przed zmianami stanu,
- jawne allowlisty dozwolonych akcji,
- testy zarówno dla przypadków allowed, jak i denied.

Unikaj:

- polegania tylko na frontend route guardach,
- ufania `userId` z request body,
- ufania `role` z klienta,
- ufania `confirmed=true`,
- chronienia tylko pierwszego kroku workflow,
- zakładania, że użytkownicy podążają zamierzonym flow UI,
- używania obscurity jako access control.

## 10. Wynik review

Funkcja powinna przejść review A01 tylko wtedy, gdy:

- nieautoryzowani użytkownicy nie mogą uzyskać dostępu do chronionych zasobów,
- nieautoryzowani użytkownicy nie mogą wykonać chronionych akcji,
- bezpośrednie requesty są odrzucane,
- finalne kroki workflow są chronione,
- stan pozostaje bez zmian po odrzuconych requestach,
- uprawnieni użytkownicy nadal mogą wykonać dozwolony workflow,
- istnieją testy regresji dla pozytywnych i negatywnych przypadków access control.
