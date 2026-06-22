# OWASP Top 10 2025 Learning Summary

This note captures the main takeaways from the OWASP Top 10 2025 mapping sprint. The [README](README.md) stays the index for the directory; this page is the reflective recap.

The sprint was built around a simple goal: move from theory to repeatable review habits. For each category, I wanted to understand the root cause, the user-controlled input path, the backend enforcement point, the safe testing approach, the impact, and the regression check that should follow the fix.

## What The Sprint Covered

The work covered all OWASP Top 10 2025 categories:

- A01 Broken Access Control: authorization mistakes, IDOR-style thinking, and trust-boundary checks.
- A02 Security Misconfiguration: debug exposure, verbose errors, unsafe defaults, and environment-specific configuration.
- A03 Software Supply Chain Failures: dependency trust, lifecycle scripts, lockfiles, SBOMs, CI/CD integrity, and unsafe install patterns.
- A04 Cryptographic Failures: sensitive data handling, weak token design, encoding versus encryption, hashing, and cookie security.
- A05 Injection: SQL injection, NoSQL injection, OS command injection, SSTI, XSS mapping, and prompt injection awareness.
- A06 Insecure Design: business logic flaws, unsafe assumptions, abuse cases, threat modelling, and STRIDE review.
- A07 Authentication Failures: password reset logic, MFA enforcement, session state, enumeration, and authentication lifecycle checks.
- A08 Software or Data Integrity Failures: integrity of data, updates, packages, artefacts, build outputs, and trusted workflows.
- A09 Security Logging and Alerting Failures: security events, audit logs, alerting, log safety, detection value, and incident response readiness.
- A10 Mishandling of Exceptional Conditions: error handling, fail-open behaviour, unsafe fallbacks, retries, timeouts, and invalid states.

## The Reusable Review Pattern

The most useful habit was repeatedly asking the same questions during review:

- What does the user control?
- Where does that data go?
- Which component trusts it?
- Is the backend enforcing the rule, or is the frontend only suggesting it?
- What happens when validation, a dependency, a session, or an external service fails?
- Does the system fail closed, or does it continue in an unsafe state?
- What should be logged, alerted, and tested after the fix?

That pattern made the categories feel connected instead of separate. It also made it easier to explain impact in plain language and to turn the fix into a regression test.

## Main Takeaways

The sprint reinforced a few themes that showed up across categories:

- frontend controls are useful for UX, but security decisions must be enforced server-side,
- input must be handled according to context, not cleaned with one generic rule,
- authentication and authorization are separate concerns,
- safe design matters before implementation starts,
- logs are security assets, not only debugging output,
- dependency and CI/CD trust are part of application security,
- error handling can create security risk when exceptional states are allowed to continue unsafely,
- regression tests are part of the fix, not an optional extra.

## What I Carry Forward

The output from the sprint is intentionally defensive and developer-oriented. It is not an exploit guide and it does not include real target data, credentials, private information, or instructions for testing systems without permission.

The next areas to extend are:

1. Docker and container security.
2. CI/CD security.
3. Security review of local development workflows.
4. More realistic security finding write-ups.
5. Additional practice with secure design and threat modelling.
