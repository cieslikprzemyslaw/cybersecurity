# .NET - ryzyka związane z uploadem

Ryzyko zależy od tego, czy aplikacja używa klasycznego ASP.NET czy ASP.NET Core, a także od konfiguracji IIS i miejsca zapisu plików.

Potencjalnie niebezpieczne typy plików:

```text
.aspx
.ashx
.asmx
.config
```

## Główna lekcja

W .NET bezpośrednie wykonanie zależy od IIS i konfiguracji aplikacji, ale niebezpieczny zapis i publiczne serwowanie plików pozostają dużym ryzykiem.
