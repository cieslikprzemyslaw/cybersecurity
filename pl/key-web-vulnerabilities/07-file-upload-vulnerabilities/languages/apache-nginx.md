# Apache / Nginx - notatki o konfiguracji uploadu

Uploady są bezpieczniejsze, gdy są przechowywane poza wykonywalnym web rootem.

Ryzykowny wzorzec:

```text
/var/www/html/uploads/shell.php
```

## Główna lekcja

Konfiguracja serwera webowego może zmienić słabą funkcję uploadu w RCE.
