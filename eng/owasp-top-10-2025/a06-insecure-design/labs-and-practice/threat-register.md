# Fictional Event Booking Platform - Threat Register

This register was generated from the threat metadata stored in the supplied draw.io model.

The fields are kept in alphabetical A-Z order to match the draw.io properties:

```text
Affected element
Description
Mitigation
Severity
Status
STRIDE
STRIDE reason
Threat ID
Title
Verification
```

`Open` means the fictional threat has not been implemented and verified. It is not evidence of a vulnerability in a real production system.

## T-001 - Manipulation of authentication and role operation requests

### Affected element

API Gateway → Auth Service authentication, password-reset and role-operation requests

### Description

An attacker modifies identity, role, password-reset or account-operation values sent to the Auth Service. If client-controlled values are treated as authoritative, the attacker may affect another account or influence a privileged operation.

### Mitigation

Treat all client-supplied identity, role and reset values as untrusted. Derive the acting identity from a validated session or signed token, load roles from trusted server-side data, validate reset tokens and enforce authorization for every sensitive operation.

### Severity

High

### Status

Open

### STRIDE

Tampering

### STRIDE reason

The attacker alters security-sensitive request data as it crosses the client/server trust boundary.

### Threat ID

T-001

### Title

Manipulation of authentication and role operation requests

### Verification

Modify user ID, role, email and password-reset token values. The backend must reject unauthorised or invalid requests, make no state change and record the denied attempt.

---

## T-002 - Privilege escalation through trusted client-supplied role

### Affected element

Auth Service role and permission decisions

### Description

The Auth Service accepts or issues elevated privileges based on a client-controlled role, user ID or account value instead of trusted server-side authorization data.

### Mitigation

Ignore client-supplied roles and permissions. Load authorization data from trusted server-side sources, deny access by default, validate token integrity and require recent authentication for high-risk role changes. Target services must enforce authorization for each sensitive action.

### Severity

High

### Status

Open

### STRIDE

Elevation of Privilege

### STRIDE reason

An authenticated customer may gain administrator or other privileged capabilities beyond their intended permissions.

### Threat ID

T-002

### Title

Privilege escalation through trusted client-supplied role

### Verification

Authenticate as a normal customer and submit an administrator role or call role-management and administrator-only operations directly. The backend must return 403 and perform no state change.

---

## T-003 - Account takeover through insecure password-reset tokens

### Affected element

Auth Service password-reset workflow

### Description

An attacker obtains, predicts, replays or misuses a password-reset token to set a new password for another user's account.

### Mitigation

Generate cryptographically random high-entropy reset tokens. Bind each token to one account and one reset operation, use a short expiration time, invalidate it immediately after successful use, rate-limit attempts and prevent token leakage through logs, analytics or third-party requests.

### Severity

High

### Status

Open

### STRIDE

Spoofing

### STRIDE reason

A successful reset-token attack lets the attacker impersonate the legitimate account owner.

### Threat ID

T-003

### Title

Account takeover through insecure password-reset tokens

### Verification

Replay a used token, submit expired or modified tokens and attempt to use a token with a different account. Every attempt must be rejected without changing the password.

---

## T-004 - Account enumeration through inconsistent authentication responses

### Affected element

Auth Service → API Gateway authentication and recovery responses

### Description

An attacker compares responses for existing and non-existing usernames or email addresses during login, registration or password recovery. Differences in messages, HTTP status, response length, redirect behaviour or timing may reveal which accounts exist.

### Mitigation

Return consistent responses for existing and non-existing accounts, minimise observable timing differences and apply rate limiting, monitoring and abuse detection to repeated authentication and recovery attempts.

### Severity

Medium

### Status

Open

### STRIDE

Information Disclosure

### STRIDE reason

The application reveals account-existence information that an unauthorised party should not receive.

### Threat ID

T-004

### Title

Account enumeration through inconsistent authentication responses

### Verification

Submit equivalent login, registration and password-recovery requests for existing and non-existing accounts. Compare message, status, length, redirect behaviour and timing. Responses must not reliably reveal whether an account exists.

---

## T-005 - Account impersonation through stale or unrevoked sessions

### Affected element

Auth Service session and token lifecycle

### Description

A previously issued or stolen session remains valid after logout, password reset, password change, account disablement or a security-sensitive role change. A person possessing the session token may continue acting as the legitimate user.

### Mitigation

Define an explicit session-revocation policy. Invalidate relevant sessions after logout, reset, password change and account disablement; rotate session identifiers after authentication and privilege changes; validate expiry, account status and revocation state; allow users to revoke other active sessions.

### Severity

High

### Status

Open

### STRIDE

Spoofing

### STRIDE reason

A stale or stolen session allows an attacker to impersonate the authenticated account owner.

### Threat ID

T-005

### Title

Account impersonation through stale or unrevoked sessions

### Verification

Log in using two browsers. In the first, perform logout, password reset, password change, account disablement and a sensitive role change. Test the old session in the second browser; it must be rejected according to the defined policy.

---

## T-006 - Unauthorised management of another organizer's event

### Affected element

Event Service event-management operations

### Description

An authenticated event organizer changes an event ID and attempts to view, update, publish or delete an event owned by another organizer. If the service checks only that the user is authenticated, the organizer may operate outside their authorised scope.

### Mitigation

Derive the acting identity from a validated session or signed token. Enforce object-level authorization for every event read, update, publish and delete action. Verify ownership or an explicitly assigned management permission, deny by default and do not rely on hidden frontend controls.

### Severity

High

### Status

Open

### STRIDE

Elevation of Privilege

### STRIDE reason

The organizer attempts to extend valid permissions over their own events to events owned by another organizer.

### Threat ID

T-006

### Title

Unauthorised management of another organizer's event

### Verification

Create events under organizer accounts A and B. As A, replace event IDs with B's event ID in read, update, publish and delete requests. The backend must reject all unauthorised operations, disclose no private data and make no state change.

---

## T-007 - Disclosure of private events to unauthorised users

### Affected element

Event Service listings, search and event-detail responses

### Description

A private or restricted event appears in public search results, listings or API responses, or its details can be retrieved directly by an unauthorised user who knows or guesses the event ID.

### Mitigation

Enforce visibility and access rules in the Event Service for every listing, search and detail request. Filter unauthorised records before building responses, ensure caches and search indexes honour visibility changes and return only the minimum authorised fields.

### Severity

High

### Status

Open

### STRIDE

Information Disclosure

### STRIDE reason

Private event information is returned to a user who is not authorised to receive it.

### Threat ID

T-007

### Title

Disclosure of private events to unauthorised users

### Verification

Create a private event limited to a selected user or group. As an unauthorised user and unauthenticated visitor, test listings, search, direct detail endpoints and modified event IDs. The event and its private metadata must not be returned.

---

## T-008 - Invalid or unauthorised event state through manipulated event data

### Affected element

Event Service create and update operations

### Description

An organizer manipulates client-controlled values such as capacity, owner ID, publication status or event dates. Without server-side business invariants, the service may store an invalid or unauthorised event state.

### Mitigation

Validate event data server-side. Enforce capacity limits and valid date relationships, derive ownership from trusted identity data, model allowed publication-state transitions and reject unknown, invalid or unauthorised changes before writing to the database.

### Severity

High

### Status

Open

### STRIDE

Tampering

### STRIDE reason

The attacker alters authoritative event data and may cause the system to store an invalid or unauthorised state.

### Threat ID

T-008

### Title

Invalid or unauthorised event state through manipulated event data

### Verification

Submit negative, zero and excessively large capacity values, an end date before the start date, a modified owner ID and unauthorised publication transitions. The backend must return a controlled error and make no database change.

---

## T-009 - Unauthorised access to another customer's booking

### Affected element

Booking Service booking, cancellation and ticket operations

### Description

An authenticated customer changes a booking ID and attempts to view, update, cancel or download a ticket belonging to another customer. If only authentication is checked, the customer may operate outside their authorised scope.

### Mitigation

Derive the acting identity from a validated session or signed token. Enforce object-level authorization for every booking read, update, cancellation and ticket-download operation. Verify ownership or an explicitly authorised administrative permission and return only minimum authorised data.

### Severity

High

### Status

Open

### STRIDE

Elevation of Privilege

### STRIDE reason

The customer attempts to extend permissions from their own bookings to a booking owned by another user.

### Threat ID

T-009

### Title

Unauthorised access to another customer's booking

### Verification

Create bookings with customer accounts A and B. As A, replace the booking ID with B's in read, update, cancellation and ticket-download requests. The backend must reject each attempt, return no booking data and make no state change.

---

## T-010 - Invalid booking and ticket state transitions

### Affected element

Booking Service booking and ticket lifecycle

### Description

A customer modifies requests, skips steps, changes their order or replays operations to confirm a pending booking, obtain a valid ticket before payment, reactivate a cancelled booking or continue using a ticket after cancellation or refund.

### Mitigation

Implement an explicit server-side state machine for bookings and tickets. Permit only defined state transitions, derive payment state from trusted Payment Service data, validate the current state before every transition and atomically invalidate tickets after cancellation or refund.

### Severity

High

### Status

Open

### STRIDE

Tampering

### STRIDE reason

The attacker alters authoritative booking or ticket state, or forces the workflow into a state the business rules do not permit.

### Threat ID

T-010

### Title

Invalid booking and ticket state transitions

### Verification

Attempt to skip payment, reorder confirmation and payment steps, replay confirmation requests, reactivate a cancelled booking and reuse a ticket after cancellation or refund. Every invalid transition must be rejected without changing state.

---

## T-011 - Overbooking through concurrent booking requests

### Affected element

Booking Service, Event Service and booking-capacity data flow

### Description

Multiple simultaneous requests reserve the same final places because availability is checked and updated non-atomically. The system may confirm more tickets than the event capacity permits.

### Mitigation

Enforce the capacity invariant in a transaction or other atomic concurrency control. Use database constraints, locking or compare-and-set semantics where appropriate. Make booking creation idempotent and re-check availability at the authoritative write boundary.

### Severity

High

### Status

Open

### STRIDE

Tampering

### STRIDE reason

Concurrent requests cause unauthorised changes to booking and capacity state, breaking the integrity of the event inventory.

### Threat ID

T-011

### Title

Overbooking through concurrent booking requests

### Verification

Create an event with one remaining place and send multiple booking requests concurrently from separate sessions. At most one request may succeed, capacity must never become negative and no duplicate confirmed tickets may be issued.

---

## T-012 - Payment amount manipulation through untrusted booking data

### Affected element

Booking Service → Payment Service create-payment and refund requests

### Description

A customer manipulates amount, currency, booking ID or refund value in a request. If the Payment Service trusts values originating from the client or an unverified upstream request, an incorrect payment or refund may be processed.

### Mitigation

Calculate payment and refund amounts from trusted server-side booking, pricing and policy data. Bind payment records to the authenticated customer and booking, validate currency and state, and re-calculate the authoritative amount before creating a payment or refund.

### Severity

High

### Status

Open

### STRIDE

Tampering

### STRIDE reason

The attacker alters financial values used by the payment workflow.

### Threat ID

T-012

### Title

Payment amount manipulation through untrusted booking data

### Verification

Modify amount, currency, booking ID and refund value in intercepted requests. The Payment Service must ignore or reject manipulated values and use only the authoritative server-calculated amount.

---

## T-013 - False payment confirmation through forged or replayed webhook

### Affected element

External Payment Provider → Payment Service webhook flow

### Description

An attacker sends a forged payment-status callback or replays a previously valid webhook. If authenticity and freshness are not verified, a booking may be marked paid or refunded without a legitimate provider event.

### Mitigation

Verify provider signatures using the raw payload, validate timestamp and event type, reject stale messages, store provider event IDs and process each event once. Bind the event to the expected payment, booking, amount and currency before changing state.

### Severity

High

### Status

Open

### STRIDE

Spoofing

### STRIDE reason

The attacker impersonates the external payment provider when submitting a false callback.

### Threat ID

T-013

### Title

False payment confirmation through forged or replayed webhook

### Verification

Send unsigned, incorrectly signed, modified, expired and replayed webhook payloads. No payment, refund, booking or ticket state may change unless the event is authentic, current, expected and not previously processed.

---

## T-014 - Duplicate payment or refund through repeated operations

### Affected element

Payment Service payment, refund and Payment Database operations

### Description

Retries, replayed requests or concurrent operations create the same payment or refund more than once because the workflow lacks idempotency and state validation.

### Mitigation

Require unique idempotency keys for payment and refund creation, persist them with unique constraints, model allowed financial state transitions and make repeated identical requests return the original result without executing the operation again.

### Severity

High

### Status

Open

### STRIDE

Tampering

### STRIDE reason

Repeated requests cause duplicate or inconsistent changes to authoritative financial records.

### Threat ID

T-014

### Title

Duplicate payment or refund through repeated operations

### Verification

Repeat and concurrently submit identical payment and refund requests using the same business operation. Exactly one financial operation may occur and all retries must return a consistent existing result.

---

## T-015 - Sensitive booking or payment data sent to the wrong recipient

### Affected element

Notification Service recipient and template-data handling

### Description

A recipient address, booking reference or template variable is taken from untrusted input or stale mappings, causing ticket, booking or payment information to be sent to an unauthorised person.

### Mitigation

Derive recipients from authoritative account and booking records, authorise notification requests, use approved templates with controlled variables, minimise included personal data and validate that each notification belongs to the intended recipient and workflow state.

### Severity

High

### Status

Open

### STRIDE

Information Disclosure

### STRIDE reason

Sensitive booking, ticket or payment information is disclosed to an unauthorised recipient.

### Threat ID

T-015

### Title

Sensitive booking or payment data sent to the wrong recipient

### Verification

Modify recipient, booking ID and template variables in notification-triggering requests. Use two customer accounts and verify that each message contains only the intended customer’s minimum authorised data.

---

## T-016 - Notification flooding through unrestricted send operations

### Affected element

Booking Service and Payment Service → Notification Service flows

### Description

An attacker or faulty retry loop repeatedly triggers notification operations, flooding recipients, consuming provider quota and preventing legitimate messages from being delivered.

### Mitigation

Expose notification operations only to authorised internal services. Apply per-event deduplication, rate limits, queue quotas and retry backoff. Use idempotency identifiers for business notifications and monitor unusual send volumes and provider failures.

### Severity

Medium

### Status

Open

### STRIDE

Denial of Service

### STRIDE reason

Repeated notification requests can exhaust service or provider capacity and impair delivery of legitimate messages.

### Threat ID

T-016

### Title

Notification flooding through unrestricted send operations

### Verification

Replay and concurrently trigger the same notification event many times. The system must deduplicate the business event, enforce limits and preserve capacity for legitimate messages.

---

## T-017 - Excessive sensitive data returned in API responses

### Affected element

Backend services → API Gateway → Web Frontend response flows

### Description

API responses include fields that the current user or frontend does not need, such as internal identifiers, private attendee data, role details, payment metadata or hidden event information.

### Mitigation

Authorise before serialization, use explicit response DTOs or field allowlists, return the minimum data required for the current role and action, and prevent frontend filtering from being treated as a security control.

### Severity

High

### Status

Open

### STRIDE

Information Disclosure

### STRIDE reason

The backend returns sensitive fields to a client that is not authorised or does not need to receive them.

### Threat ID

T-017

### Title

Excessive sensitive data returned in API responses

### Verification

Capture responses as customer, organizer, administrator and unauthenticated visitor. Compare fields and test direct API calls. Each role must receive only explicitly authorised and necessary data.

---

## T-018 - Privilege escalation through inconsistent authorization across endpoints

### Affected element

API Gateway and all target service authorization boundaries

### Description

A sensitive action is protected on one route but remains accessible through an alternate endpoint, API version, batch operation or direct service path with weaker authorization enforcement.

### Mitigation

Define a central authorization policy and enforce it independently in every target service. Deny by default, maintain an endpoint-to-permission matrix, cover alternate and legacy routes, and add automated negative authorization tests for every sensitive operation.

### Severity

High

### Status

Open

### STRIDE

Elevation of Privilege

### STRIDE reason

A user reaches a privileged action through a less-protected path and gains capabilities outside their role.

### Threat ID

T-018

### Title

Privilege escalation through inconsistent authorization across endpoints

### Verification

For each sensitive operation, test normal, alternate, legacy, batch and direct API routes using unauthorised roles and modified resource IDs. Every path must return 403 and make no state change.
