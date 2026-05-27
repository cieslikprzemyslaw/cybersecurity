# Example Security Finding: Public Debug Endpoint Exposes Application Configuration and Secret Key

## Summary

A public debug endpoint exposed detailed PHP and server configuration information through a `phpinfo()` page.

The endpoint was discoverable from an HTML comment and was accessible without authentication or authorization. The exposed debug page disclosed sensitive technical information, including a sensitive application secret.

## Affected Area

```text
GET /cgi-bin/phpinfo.php
```

Discovery clue found in page source:

```html
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

## OWASP Mapping

```text
OWASP Top 10 2025: A02 - Security Misconfiguration
```

Related weakness type:

```text
Information Disclosure
Exposed Debug Functionality
Unsafe Production Configuration
Secret Exposure
```

## Risk / Impact

The exposed debug endpoint disclosed detailed application and environment configuration.

Information exposed by the page included examples such as:

- PHP version,
- operating system details,
- Server API,
- PHP configuration paths,
- loaded configuration files,
- PHP modules/extensions,
- PHP settings,
- environment/configuration values,
- sensitive application secret.

The sensitive secret value is intentionally redacted:

```text
<redacted-secret-key>
```

Exposing this type of information increases risk because an attacker can use it for reconnaissance and targeted exploitation.

If a secret key is exposed, the potential impact can be higher. Depending on how the application uses the key, an attacker may be able to forge or manipulate trusted values such as signed cookies, session data, CSRF tokens, JWTs, reset tokens, or other protected application data.

The exact impact depends on the application implementation, but exposed secrets should be treated as compromised and rotated.

## Root Cause

Development/debug functionality was deployed or left accessible in a public environment.

The debug link was hidden in an HTML comment, but the backend endpoint remained publicly reachable.

The application relied on obscurity rather than removing or properly protecting the diagnostic page.

## Evidence

A public page contained the following HTML comment:

```html
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

A user could then access the debug endpoint directly:

```http
GET /cgi-bin/phpinfo.php
```

The endpoint returned a `phpinfo()` page containing detailed PHP/server configuration and a sensitive `SECRET_KEY`.

Sensitive values are redacted in this report.

## Why This Is Security Misconfiguration

This issue is Security Misconfiguration because diagnostic functionality intended for development or troubleshooting was publicly accessible.

The issue is not XSS, SQL Injection, or Broken Access Control as the main category. The main problem is unsafe production configuration and excessive information exposure.

## Expected Secure Behaviour

In a production environment, this endpoint should not be publicly accessible.

Preferred behaviour:

```http
HTTP/1.1 404 Not Found
```

because the debug file should not be deployed at all.

If diagnostic functionality is intentionally required, it should be restricted to trusted internal/admin access only. In that case, unauthorised users should receive:

```http
HTTP/1.1 403 Forbidden
```

The response must not expose:

- `phpinfo()`,
- PHP version,
- internal paths,
- environment variables,
- configuration values,
- secrets,
- debug output.

## Remediation

Recommended remediation:

1. Remove `phpinfo.php` and other debug files from production deployments.
2. Block public access to known debug paths at the web server or reverse proxy level.
3. Remove debug references from HTML comments, templates, JavaScript bundles, and public assets.
4. Rotate the exposed `SECRET_KEY` because it should be treated as compromised.
5. Review application logs, deployment artifacts, and repository history for additional secret exposure.
6. Add secret scanning to CI/CD.
7. Add deployment checks to prevent debug files from being included in production artifacts.
8. Ensure diagnostic tools are available only in trusted internal environments, if needed at all.
9. Review environment separation between development, staging, and production.
10. Document production hardening requirements.

## Regression Test Ideas

Add tests or deployment checks to verify:

```text
GET /cgi-bin/phpinfo.php returns 404 or 403 in production-like environments.
The response does not contain "phpinfo".
The response does not contain "PHP Version".
The response does not contain "SECRET_KEY".
The response does not contain environment variables.
The production build artifact does not include phpinfo.php.
Public HTML source does not contain debug endpoint references.
Secret scanning passes before deployment.
Previously exposed secrets have been rotated.
```

## Developer Takeaway

Hiding a debug link in an HTML comment is not a security control.

Anything sent to the browser should be treated as visible to the user. The real fix is to remove or protect the backend endpoint and ensure debug functionality is not exposed in production.

Frontend can support security by avoiding leakage in HTML, JavaScript, comments, and source maps, but production security must be enforced through backend/server configuration, deployment controls, and secret management.

## Portfolio Note

This finding shows a practical AppSec workflow:

```text
HTML source review
    ↓
Hidden debug endpoint discovery
    ↓
Direct endpoint access
    ↓
Sensitive configuration leak
    ↓
Impact analysis
    ↓
Remediation and regression test planning
```
