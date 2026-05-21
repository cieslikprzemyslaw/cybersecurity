# Java - ryzyka związane z uploadem

Nowoczesne aplikacje Java/Spring zwykle obsługują requesty przez kontrolery. Uploadowane pliki zazwyczaj nie wykonują się tylko dlatego, że istnieją na dysku.

Ryzykowne przypadki mogą obejmować upload `.jsp` do lokalizacji dostępnej przez web i wykonywalnej przez serwer, zależnie od konfiguracji Tomcata albo serwera aplikacyjnego.

## Główna lekcja

W Javie RCE przez upload zależy od konfiguracji, ale path traversal, stored XSS i niebezpieczne serwowanie plików nadal są istotne.
