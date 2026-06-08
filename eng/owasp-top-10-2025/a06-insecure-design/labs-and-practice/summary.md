# A06 Practice Summary

## Three different design problems

| Exercise | Unsafe design decision | Evidence | Required design change |
|---|---|---|---|
| Client-controlled price | The client supplied an authoritative financial value | Manipulated price persisted and checkout succeeded | Calculate price and total from trusted server-side data |
| Coupon workflow | The rule was not enforced across the full promotion state | Alternating valid codes reduced the order to zero | Model usage scope, history and combinations server-side |
| Event platform threat model | Security assumptions were not yet expressed as requirements | Abuse cases exposed missing decisions before implementation | Add explicit authorization, state, idempotency and trust-boundary requirements |

## Shared mental model

```text
What must always be true?
  -> What assumption could break it?
  -> Which actor can challenge that assumption?
  -> Where should the control be enforced?
  -> How can the result be verified?
```

## Frontend takeaway

The frontend should still:

- hide unavailable actions,
- validate expected input,
- provide early feedback,
- reduce accidental mistakes.

The backend must independently protect:

- authorization,
- ownership,
- prices and totals,
- workflow state,
- private data,
- replay and concurrency-sensitive operations.

## Prevention, detection and defence in depth

### Prevention

Change the design so the unsafe outcome cannot occur.

Examples:

- calculate prices server-side,
- enforce a state machine,
- verify ownership,
- process a webhook once.

### Detection

Record suspicious or denied behaviour.

Examples:

- repeated role manipulation,
- replayed webhook event IDs,
- excessive reset attempts.

### Defence in depth

Reduce likelihood or impact in addition to the primary control.

Examples:

- rate limiting,
- recent authentication,
- monitoring,
- least privilege,
- human approval.

Logging or warnings do not replace prevention when the workflow itself is unsafe.
