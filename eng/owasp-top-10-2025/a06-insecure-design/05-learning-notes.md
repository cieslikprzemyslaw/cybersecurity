# A06: Insecure Design - Learning Notes

## Initial mental model

At the beginning, I described Insecure Design too broadly as an application that was not secured correctly.

The more useful definition is:

> The design is missing an effective control, uses an unsafe assumption or permits a workflow that should not be possible.

A lack of evidence is not automatically a confirmed design flaw. It is an unverified assumption until architecture, code or behaviour proves more.

## Corrections that mattered

### Design versus implementation

I learned that code can be technically correct and still implement an insecure design.

A developer can correctly accept a `price` field, store it and process checkout. The design is still unsafe if the client is allowed to supply the authoritative price.

### Assets versus components

I initially listed Auth Service, Booking Service and databases as assets.

The components process or store assets. The assets are the valuable identities, roles, bookings, tickets, event visibility, payments and business state.

### Facts versus assumptions

A developer saying that a reset token is "probably single-use" is not verification.

A useful review records:

- what is known,
- what is assumed,
- what could happen if the assumption is wrong,
- what the system must enforce,
- how to test it.

## Frontend perspective

Conditional rendering is real and useful:

```tsx
{isAdmin && <AdminPanel />}
```

The component is not rendered when the condition is false. This improves UX and reduces accidental actions.

It is not authorization because the user may still:

- call the API directly,
- modify a request,
- change browser state,
- receive sensitive data already returned by the API.

My frontend experience is useful for identifying the browser/API trust boundary, client-controlled state and places where teams may confuse presentation with enforcement.

## Business logic lessons

The first lab used an obviously dangerous client-controlled value.

The second lab was subtler. Every coupon was valid and every individual request was accepted. The vulnerability appeared only through the sequence:

```text
NEWCUST5 -> SIGNUP30 -> NEWCUST5 -> SIGNUP30
```

This showed why payload guessing and input validation are not enough for business logic. The reviewer must understand state, limits, order and repetition.

## Threat-modelling lessons

The useful part of threat modelling was not drawing every possible component.

The useful sequence was:

```text
asset
  -> actor and trust boundary
  -> unsafe assumption
  -> abuse case
  -> missing control
  -> security requirement
  -> verification
```

STRIDE helped classify threats, but selecting a category required identifying the main security effect:

- changed authoritative data: Tampering,
- impersonated identity or provider: Spoofing,
- leaked private data: Information Disclosure,
- expanded permissions: Elevation of Privilege,
- exhausted service capacity: Denial of Service.

## Tooling lesson

A large Threat Dragon diagram became difficult to read. Draw.io worked better for the full system context and DFD.

Threat Dragon remains useful for smaller workflows and built-in threat fields, but the architecture should be divided when the diagram becomes dense.

## Final takeaway

I do not need to work as a senior security architect to participate in a useful design review.

At my current FE-to-AppSec level, I should be able to:

- identify the asset and business rule,
- treat the browser as untrusted,
- challenge assumptions,
- ask where authorization and state validation happen,
- propose a testable security requirement,
- design a negative or abuse-case test,
- distinguish prevention from monitoring.
