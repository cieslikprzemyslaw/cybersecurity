# A06: Insecure Design Checklist

Use this checklist during feature design, architecture review, code review planning or authorised testing.

## Business requirement and invariants

- What business rule is the feature implementing?
- What must always be true?
- What must never happen?
- Which outcome would create financial, privacy, integrity or availability impact?
- Is the rule specific enough to be tested?
- Is the rule enforced by the system or only described in UI text?

## Assets and actors

- What data, identity, functionality or business process is valuable?
- Who are the actors?
- Which actors are authenticated, privileged, external or anonymous?
- Which actor has the highest impact if their account is compromised?
- Are administrative roles separated by responsibility and least privilege?

## Entry points and data flows

- Where can data or commands enter?
- Which values are supplied by the browser, mobile app, API client or third party?
- Where does each value travel?
- Which component makes the final security decision?
- Is an intermediate service trusted more than it should be?
- Does the target service independently verify the operation?

## Trust boundaries

- Where does trust change between client and server?
- Where does trust change between internal services?
- Where does data cross into or from a third-party provider?
- Are callbacks, webhooks and messages authenticated?
- Is frontend state treated as user-controlled?
- Can a direct API request bypass the intended UI flow?

## Frontend review

- Is a hidden button being treated as authorization?
- Is a route guard the only protection for a privileged feature?
- Can disabled or hidden values be modified in the request?
- Does the frontend send authoritative price, role, ownership or status values?
- Does the API return data that the frontend merely filters out?
- Are client-side controls retained for UX without being mistaken for enforcement?

## Authentication, roles and sessions

- Is identity derived from a validated session or signed token?
- Can the client supply or change its role?
- Does each target service enforce authorization?
- Are password-reset tokens random, short-lived, single-use and account-bound?
- Are authentication and recovery responses resistant to enumeration?
- Which sessions are invalidated after logout, password reset, password change, account disablement and role change?
- Do sensitive role changes require recent authentication?

## Object ownership and privacy

- Is ownership checked on every read, update, delete and download action?
- Are alternate, batch and legacy routes covered?
- Are private records filtered before response serialization?
- Do search, listing, detail, export and cache paths use the same access rules?
- Are response fields allowlisted for the current role and operation?

## Workflow and state

- Can a step be skipped?
- Can actions be reordered?
- Can the same action be repeated?
- Can a token, coupon, webhook, payment or refund be replayed?
- Are allowed state transitions modelled explicitly?
- Is the current authoritative state checked before every transition?
- What happens after cancellation, expiry, refund or account disablement?
- Can simultaneous requests violate capacity, balance or uniqueness rules?

## Payments and external providers

- Is the amount calculated from trusted server-side data?
- Are currency, booking, customer and refund values bound together?
- Are payment and refund operations idempotent?
- Are webhook signatures verified using the expected raw payload?
- Are timestamp, event ID, event type, amount and currency checked?
- Are replayed or stale provider events rejected?
- Does an external provider have only the minimum required access?

## Failure behaviour

- Does the system fail closed when a dependency times out?
- Can an error path bypass authorization or state validation?
- What happens after a partial failure between services?
- Can retries duplicate a payment, refund, booking or notification?
- Are unknown states rejected rather than silently accepted?
- Is recovery safe and idempotent?

## Security requirements

A useful security requirement should be:

- specific,
- testable,
- connected to an asset and realistic threat,
- independent of the frontend,
- implementation-independent where possible.

Example:

> The Booking Service must verify booking ownership for every read, update, cancellation and ticket-download operation.

## Verification

- What negative test proves the control works?
- What state must remain unchanged after rejection?
- Which response and audit evidence should be produced?
- Does the test cover direct API access rather than only the UI?
- Does the test cover replay, reordering or concurrency when relevant?
