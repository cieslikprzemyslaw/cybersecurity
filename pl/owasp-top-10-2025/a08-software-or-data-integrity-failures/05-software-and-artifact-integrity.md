# Integralność software i artefaktów

## Aktualizacje oprogramowania

Update nie powinien być instalowany tylko dlatego, że został pobrany ze znajomego URL.

Przegląd obejmuje:

- autentyczność źródła,
- podpis cyfrowy,
- zaufany klucz weryfikacyjny,
- reguły wersji i rollback,
- świeżość lub czas ważności,
- weryfikację przed instalacją,
- zachowanie fail-closed,
- logowanie błędów.

## Artefakty builda i deploymentu

Artefakt zbudowany w CI powinien pozostać tym samym artefaktem zatwierdzonym do deploymentu.

Pytania:

- Kto może uruchomić lub zmienić build?
- Kto może podmienić artefakt po buildzie?
- Czy provenance jest zapisane?
- Czy artefakt jest podpisany?
- Czy podpis jest sprawdzany przed deploymentem?
- Czy niezaufane joby mają dostęp do signing keys lub production secrets?
- Czy permissions build i deployment są rozdzielone?
- Czy wdrażany digest odpowiada zatwierdzonemu digestowi?

To jest świadomość A08, a nie pełny projekt bezpieczeństwa CI/CD.

## Zewnętrzne skrypty przeglądarkowe

Third-party script wykonuje się w kontekście aplikacji i może mieć kontakt z:

- DOM,
- formularzami,
- odpowiedziami API widocznymi dla JavaScript,
- storage dostępnym dla JavaScript,
- działaniami użytkownika,
- logiką aplikacji.

Załadowanie skryptu oznacza rozszerzenie zaufania na jego źródło i ścieżkę dostawy.

Przegląd:

- Czy provider jest zatwierdzony?
- Czy URL jest kontrolowany?
- Czy wersja jest przypięta?
- Czy zawartość jest wystarczająco statyczna dla SRI?
- Czy używany jest restrykcyjny Content Security Policy?
- Do jakich danych skrypt ma dostęp?
- Jaki jest fallback po kompromitacji albo awarii dostawcy?

## Subresource Integrity

SRI pozwala przeglądarce porównać pobrany zasób z oczekiwanym hashem:

```html
<script
  src="https://cdn.example/library.js"
  integrity="sha384-EXPECTED_HASH"
  crossorigin="anonymous">
</script>
```

Jeśli pobrane bytes nie pasują, przeglądarka blokuje zasób.

SRI pomaga dla statycznych zasobów zewnętrznych, ale nie:

- weryfikuje paczek npm podczas buildu,
- zastępuje przeglądu dostawcy,
- chroni dynamiczne skrypty, których zawartość celowo się zmienia,
- dowodzi, że zatwierdzona wersja była bezpieczna,
- chroni proces, który zatwierdził złośliwy hash.

## Artefakty builda frontendowego

Przydatne kontrole:

- powtarzalna instalacja z lockfile i `npm ci`,
- zapisane digests artefaktów,
- niezmienne przechowywanie artefaktów,
- promocja tego samego artefaktu między środowiskami,
- ograniczony write access,
- provenance lub signing adekwatne do ryzyka,
- weryfikacja przed deploymentem.

Przypięta wersja poprawia reproducibility. Sama nie dowodzi, że pobrane bytes są autentyczne i niezmienione.

## Konfiguracja i stan klienta

Traktuj jako niezaufane, dopóki zaufany komponent ich nie zweryfikuje:

- cookies,
- hidden inputs,
- URL parameters,
- `localStorage`,
- `sessionStorage`,
- JSON configuration pobierane z edytowalnej lokalizacji,
- feature flags przesłane przez przeglądarkę,
- role lub uprawnienia,
- ceny produktów,
- identyfikatory kont,
- konfiguracja CMS renderowana we frontendzie.

Serwer powinien wyprowadzać wrażliwe decyzje ze swojego zaufanego stanu.
