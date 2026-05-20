# .NET Upload Risks

Risk depends on classic ASP.NET vs ASP.NET Core, IIS configuration and storage location.

Potentially dangerous file types can include:

```text
.aspx
.ashx
.asmx
.config
```

## Main Takeaway

In .NET, direct execution depends on IIS and application configuration, but unsafe storage and public serving remain major risks.
