# A06: Insecure Design - Overview

## What is Insecure Design?

Insecure Design exists when a system or workflow is designed without an effective control needed to protect an asset or business rule.

The problem may be:

- a missing security control,
- a control placed at the wrong trust boundary,
- an unsafe assumption about user behaviour,
- an incomplete workflow or state model,
- a control that cannot protect the intended operation.

Correctly written code can still implement an insecure design exactly as specified.

## Core mental model

```text
business requirement
  -> design decision
  -> trust assumption
  -> security control
  -> expected and unexpected behaviour
```

Useful review questions:

- What is the system trying to protect?
- Who are the actors?
- What can each actor do?
- Which inputs and state are controlled by the client?
- Where does trust change?
- What must always be true?
- What must never happen?
- What if a step is skipped, repeated, reordered or executed concurrently?
- What control must exist before implementation starts?
- How will the control be verified?

## Important distinctions

| Problem | Meaning | Example |
|---|---|---|
| Insecure Design | The design lacks an effective control or permits an unsafe workflow | The client supplies the authoritative product price |
| Implementation defect | The correct control was designed, but the code implements it incorrectly | An ownership check uses the wrong comparison or resource identifier |
| Security Misconfiguration | A suitable mechanism exists, but its settings are unsafe | Authentication is disabled in production through configuration |
| Broken Access Control | A user can access data or actions outside their permission | An organizer changes `eventId` and edits another organizer's event |

Broken Access Control may result from a design flaw or an implementation defect. The observed behaviour alone does not always prove which lifecycle stage introduced the issue.

A missing test or missing evidence is not automatically proof of Insecure Design. It means the control is not yet verified.

## Frontend and the trust boundary

Frontend controls are useful for:

- usability,
- early feedback,
- preventing accidental mistakes,
- hiding unavailable actions,
- defence in depth.

They are not the primary enforcement boundary.

For example:

```tsx
{isAdmin && <AdminPanel />}
```

When `isAdmin` is false, React does not render the component into the DOM. This improves the interface, but it does not prove that the user cannot call the underlying API directly.

The backend must still:

- derive identity from a validated session or token,
- enforce role and resource ownership,
- validate workflow state,
- calculate authoritative values from trusted data,
- return only authorised data.

A React route guard, disabled input or hidden button controls presentation. Server-side authorization controls capability.

## Facts, assumptions, risks and requirements

### Fact

Something supported by architecture, code, a request/response, documented policy or observed behaviour.

### Assumption

Something believed to be true but not yet confirmed.

### Risk

What could happen if an assumption is wrong or a control fails.

### Security requirement

A specific and testable rule the system must enforce.

Example:

```text
Fact:
The checkout request contains a client-controlled price.

Assumption:
The backend recalculates the price.

Risk:
The client may purchase a product for a manipulated amount.

Security requirement:
The server must calculate the product price and order total from trusted server-side product data.
```

## Business logic and state

Many A06 issues use individually valid values and actions.

The unsafe outcome appears through:

- repeated actions,
- reordered steps,
- skipped steps,
- invalid state transitions,
- stale state,
- concurrent requests,
- trusting values calculated by the client,
- inconsistent rules across routes.

Input validation cannot fix a workflow whose allowed state transitions are wrong.

## Secure design principles used in this module

- Server-side enforcement
- Deny by default
- Least privilege
- Secure defaults
- Explicit state machines
- Trusted server-side calculations
- Object-level authorization
- Idempotency
- Replay protection
- Fail-safe behaviour
- Minimal data disclosure
- Defence in depth
- Human approval or recent authentication for high-risk actions

## Practical examples completed

- A checkout trusted a client-controlled product price.
- Alternating valid coupon codes bypassed the intended usage limit.
- A fictional Event Booking Platform was modelled with actors, assets, data flows, trust boundaries, security assumptions and 18 prioritised threats.

Detailed evidence is kept in `labs-and-practice/` so this overview remains concise.
