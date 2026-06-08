# Fictional Event Booking Platform Threat Model

## Purpose

This exercise used a fictional system so the model could be shared publicly without exposing employer, customer or proprietary architecture.

The goal was to practise lightweight design review rather than produce a complete enterprise architecture.

## Tool choice

A single large OWASP Threat Dragon diagram became difficult to read.

Draw.io was used for:

- system context,
- the application DFD,
- trust boundaries,
- threat markers on a separate layer,
- custom threat metadata,
- a threat-register page.

Threat Dragon can still be useful for smaller workflow-specific diagrams, but the full platform was clearer in draw.io.

## System context

Actors:

- Customer
- Event Organizer
- Platform Administrator
- External Payment Provider
- External Email Provider

Main internal components:

- Web Frontend
- API Gateway
- Auth Service
- Event Service
- Booking Service
- Payment Service
- Notification Service
- User, Event, Booking and Payment databases

## Main trust boundaries

### Client / Server Trust Boundary

Everything originating from the browser is treated as modifiable:

- IDs,
- roles,
- prices,
- quantities,
- state fields,
- route order,
- replayed requests.

### Internal / Third-Party Trust Boundary

Payment and email providers are external dependencies.

Callbacks and delivery events must be authenticated, validated and replay-resistant before they influence internal state.

## Key assets

- User identities, sessions, roles and account recovery
- Event ownership, visibility, capacity and publication state
- Bookings, ticket validity and attendee data
- Payment and refund state
- Notification recipient and content confidentiality
- Availability of booking, payment and notification workflows

## Lightweight workflow

1. Identify assets.
2. Identify actors.
3. Map entry points and data flows.
4. Mark trust boundaries.
5. Record security assumptions.
6. Create realistic abuse cases.
7. Identify existing and missing controls.
8. Write testable security requirements.
9. Add verification tests.
10. Track status and ownership.

## STRIDE use

STRIDE was used as a supporting classification, not as the whole exercise.

Each threat includes:

- one main STRIDE category,
- a short reason explaining why it applies,
- affected element,
- description,
- mitigation,
- severity,
- status,
- verification.

Examples:

- `Spoofing`: stolen session or forged provider webhook.
- `Tampering`: manipulated event state, booking state or payment amount.
- `Information Disclosure`: private event or excessive API data.
- `Elevation of Privilege`: accessing another user's booking or event.
- `Denial of Service`: notification flooding.

## Threat scope

The final model contains 18 prioritised threats:

- 5 authentication and session threats,
- 3 event-management threats,
- 3 booking and ticket threats,
- 3 payment and webhook threats,
- 2 notification threats,
- 2 cross-cutting API and authorization threats.

The full details are in [threat-register.md](threat-register.md).

## Status meaning

All threats are marked `Open`.

This means the fictional controls have not been implemented and verified. It is not a claim that a real production application contains these vulnerabilities.

## Visual file

[fictional-event-platform-complete-threat-model.drawio](fictional-event-platform-complete-threat-model.drawio)

The file contains:

- `01-System-Context`
- `02-Application-DFD`
- `03-Threat-Register`
- an `Architecture` layer
- a `Threats` layer
- metadata for `T-001` through `T-018`

## Main learning outcome

The most useful threat-model entry was not the longest one.

A useful entry connected:

```text
asset
  -> unsafe assumption
  -> abuse case
  -> missing control
  -> testable security requirement
  -> verification
```
