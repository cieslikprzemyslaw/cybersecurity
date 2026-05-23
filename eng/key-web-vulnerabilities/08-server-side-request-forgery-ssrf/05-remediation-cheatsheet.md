# SSRF Remediation Cheatsheet

## Purpose

Use this note when thinking about how to prevent SSRF as a developer or how to write remediation advice in a finding.

SSRF fixes should focus on controlling where the backend is allowed to send requests.

## Core Principle

Do not let users control arbitrary backend request destinations.

If the application needs to fetch external or internal resources, the allowed destinations should be tightly controlled and validated.

## Safer Design Options

### Prefer IDs Over URLs

Instead of accepting:

```json
{
  "stockApi": "http://internal-stock-api/product/stock/check"
}
```

Prefer accepting a safe identifier:

```json
{
  "storeId": "glasgow-01"
}
```

Then map that identifier server-side to a known safe backend endpoint.

### Use Server-Side Mappings

Example:

```text
storeId=1 -> http://stock-service-1.internal
storeId=2 -> http://stock-service-2.internal
```

The user controls only the ID, not the URL.

### Use Strict Allowlists

If URLs must be accepted, allow only specific trusted destinations.

Validate:

- scheme,
- hostname,
- resolved IP,
- port,
- path if needed,
- redirects,
- final destination.

Avoid loose checks such as:

```text
url contains "trusted-site.com"
url starts with "https://trusted-site.com"
```

These can be bypassed with URL parser tricks.

## Block Dangerous Destinations

The backend should not be allowed to request:

```text
localhost
127.0.0.0/8
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
169.254.0.0/16
169.254.169.254
internal admin hosts
```

Also consider IPv6 equivalents such as:

```text
::1
fc00::/7
fe80::/10
```

## Validate the Final Destination

Validation should happen after:

- URL parsing,
- URL decoding,
- DNS resolution,
- redirects,
- canonicalisation / normalisation.

Do not validate only the raw input string.

Important question:

```text
Where will the HTTP client actually connect?
```

That final destination is what must be safe.

## Handle Redirects Carefully

If redirects are allowed:

- validate every redirect hop,
- re-resolve and re-check the final host/IP,
- block redirects to internal ranges,
- limit redirect count,
- avoid protocol downgrade or unexpected protocol switching.

If redirects are not required, disable them.

## Restrict Protocols and Ports

Allow only required schemes:

```text
https
http only if genuinely needed
```

Block unexpected schemes such as:

```text
file
gopher
ftp
dict
ldap
```

Restrict ports to expected values:

```text
443
80 if needed
specific known internal service ports only if required
```

## Network-Level Defences

Application-level validation is not enough.

Use infrastructure controls:

- egress firewall rules,
- network segmentation,
- service mesh policies,
- metadata endpoint protections,
- cloud IAM least privilege,
- block access to internal admin services from application workloads unless required.

## Protect Cloud Metadata

If running in cloud:

- block metadata endpoint access where not required,
- use cloud provider metadata protections,
- require session tokens where supported,
- apply least privilege to instance roles,
- monitor metadata access.

High-risk endpoint:

```text
169.254.169.254
```

## Do Not Trust Internal Requests Alone

Internal admin panels must still enforce authentication and authorization.

Do not rely only on:

```text
request comes from localhost
request comes from internal IP
request comes from trusted network
```

SSRF can make malicious requests appear to come from a trusted internal source.

## Logging and Monitoring

Log suspicious backend outbound requests:

- requests to localhost,
- requests to private IP ranges,
- requests to metadata endpoints,
- unusual ports,
- many failed internal requests,
- redirect chains to unexpected hosts.

Monitoring is not a replacement for prevention, but it helps detect exploitation attempts.

## Regression Test Ideas

After fixing SSRF, add tests for:

```text
http://localhost/admin
http://127.0.0.1/admin
http://127.1/admin
http://10.0.0.1/
http://172.16.0.1/
http://192.168.0.1/
http://169.254.169.254/
encoded internal hosts
double-encoded paths
redirect to internal host
allowed-host@blocked-host
blocked-host#allowed-host
allowed-host.blocked-host
unexpected ports
```

Expected result:

```text
The backend must not send requests to blocked internal destinations.
```

## Developer-Friendly Remediation Summary

A good remediation note should say:

1. Do not accept arbitrary user-controlled URLs for backend requests.
2. Replace user-controlled URLs with server-side mappings where possible.
3. Use a strict allowlist of expected destinations.
4. Validate the final resolved destination, not only the raw input.
5. Block loopback, private, link-local and metadata IP ranges.
6. Re-check every redirect hop.
7. Restrict schemes and ports.
8. Add network egress controls.
9. Keep internal admin interfaces authenticated.
10. Add regression tests for known bypass patterns.

## Key Takeaway

SSRF remediation is about controlling backend egress.

The backend should only be able to request destinations that are explicitly required and safe.
