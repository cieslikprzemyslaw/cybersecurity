# A06: Insecure Design - Labs and Practice

This file indexes the completed legal practice. Detailed evidence is stored in the linked files rather than repeated here.

## Completed learning

- TryHackMe: OWASP Top 10 2025 - Application Design Flaws
- PortSwigger: Excessive trust in client-side controls
- PortSwigger: Flawed enforcement of business rules
- Fictional Event Booking Platform design and threat-model exercise

## Detailed practice notes

### Client-controlled product price

[Excessive trust in client-side controls](labs-and-practice/01-excessive-trust-in-client-side-controls.md)

Main lesson:

> The server trusted a client-controlled price when it should have calculated the amount from trusted product data.

### Coupon workflow abuse

[Flawed enforcement of business rules](labs-and-practice/02-flawed-enforcement-of-business-rules.md)

Main lesson:

> Two valid coupons and valid individual actions created an invalid outcome because the application did not enforce the rule across the full cart state.

### Threat modelling

[Fictional Event Booking Platform threat model](labs-and-practice/03-fictional-event-platform-threat-model.md)

The exercise covered:

- actors and assets,
- system context and application DFD,
- client/server and third-party trust boundaries,
- authentication, events, booking, payments and notifications,
- STRIDE classification with a short reason,
- mitigations and verification tests,
- 18 prioritised threats.

## Current limits

- The model is fictional and does not represent a real employer or customer system.
- Threat severity is contextual and would need business impact, data classification and deployment details in a real review.
- `Open` means the fictional control has not been implemented and verified; it does not mean a real vulnerability was discovered.
- STRIDE supports the review but does not replace business-logic analysis.
