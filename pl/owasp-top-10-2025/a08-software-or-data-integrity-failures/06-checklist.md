# Checklista A08

Używaj tylko sekcji istotnych dla analizowanej funkcji.

## Asset i granica zaufania

- [ ] Zidentyfikuj software, update, build, konfigurację, cookie, obiekt albo dane.
- [ ] Zapisz ich źródło.
- [ ] Zapisz, jak trafiają do zaufanego komponentu.
- [ ] Ustal, kto może je zmienić podczas transportu i w storage.
- [ ] Ustal, który komponent traktuje je jako autorytatywne.

## Integralność i autentyczność

- [ ] Ustal, czy wymagana jest integralność, autentyczność, poufność czy kilka właściwości.
- [ ] Sprawdź, czy weryfikacja używa zaufanego klucza lub oczekiwanego hasha.
- [ ] Sprawdź, czy atakujący może podmienić artefakt i checksum.
- [ ] Potwierdź, że weryfikacja następuje przed użyciem, wykonaniem, instalacją, deploymentem, parsowaniem lub deserializacją.
- [ ] Potwierdź, że błąd powoduje odrzucenie, a nie fallback do niezabezpieczonego przetwarzania.
- [ ] Potwierdź logowanie błędów bez ujawniania secrets.

## Stan kontrolowany przez klienta

- [ ] Sprawdź cookies, hidden fields, storage, URL parameters i wartości generowane przez frontend.
- [ ] Sprawdź, czy role, permissions, ceny, identity lub account state pochodzą z przeglądarki.
- [ ] Potwierdź, że backend wyprowadza security-sensitive state z zaufanego źródła po stronie serwera.
- [ ] Potwierdź autoryzację chronionych endpointów niezależnie od widoczności UI.
- [ ] Sprawdź expiry, context binding i replay protection dla podpisanego stanu.

## Deserializacja

- [ ] Zidentyfikuj natywną deserializację inputu kontrolowanego przez użytkownika.
- [ ] Zidentyfikuj warstwy kodowania, np. URL encoding i Base64.
- [ ] Nie uznawaj kodowania za ochronę integralności.
- [ ] Ustal typ obiektu, pola i security-sensitive attributes.
- [ ] Potwierdź weryfikację integralności przed deserializacją.
- [ ] Preferuj proste formaty i jawne schematy.
- [ ] Odrzucaj nieoczekiwane typy i pola.
- [ ] Nie polegaj tylko na walidacji po deserializacji.
- [ ] Nie deklaruj gadget chain lub RCE bez dowodów.

## Skrypty zewnętrzne i CDN

- [ ] Zrób inventory zewnętrznych skryptów.
- [ ] Potwierdź zatwierdzone źródła.
- [ ] Pinuj wersje tam, gdzie jest to odpowiednie.
- [ ] Użyj SRI dla właściwych statycznych zasobów cross-origin.
- [ ] Sprawdź CSP i ograniczenia ładowania skryptów.
- [ ] Sprawdź, do jakich danych aplikacji i użytkownika ma dostęp każdy skrypt.
- [ ] Zdefiniuj reakcję na kompromitację dostawcy.

## Aktualizacje, build i deployment

- [ ] Weryfikuj updates zaufanym podpisem przed instalacją.
- [ ] Ogranicz, kto może publikować lub zatwierdzać updates.
- [ ] Chroń signing keys przed niezaufanymi jobami.
- [ ] Zapisuj artefact provenance lub digest.
- [ ] Zapobiegaj modyfikacji między buildem a deploymentem.
- [ ] Rozdziel uprawnienia build i deployment.
- [ ] Wdrażaj ten sam zatwierdzony artefakt między środowiskami, jeśli to możliwe.
- [ ] Odrzucaj niepodpisane, nieoczekiwane lub zmienione artefakty.

## Dowody i raportowanie

- [ ] Zapisz oryginalny artefakt lub wartość.
- [ ] Zapisz pojedynczą kontrolowaną zmianę.
- [ ] Zapisz, czy aplikacja ją zaakceptowała.
- [ ] Oddziel dowód z UI od dowodu z chronionego endpointu.
- [ ] Opisz wyłącznie wykazany wpływ.
- [ ] Oddziel fakty od założeń.
- [ ] Opisz brakującą kontrolę integralności lub autentyczności.
- [ ] Napisz testowalny security requirement.
