# A06: Insecure Design - Regression Test Ideas

Regression tests should prove that the required business rule remains true when the client modifies values, bypasses the UI or uses the workflow in an unexpected order.

## Client-controlled authoritative values

- Modify a submitted product price; the server must ignore or reject it and calculate the amount from trusted product data.
- Remove the price field; the server must still calculate the correct amount.
- Modify role, owner ID, user ID, event ID and booking ID values; unauthorized actions must return `403` and make no state change.
- Modify status fields such as `published`, `paid` or `confirmed`; the client must not choose authoritative state.
- Verify that API responses do not include sensitive fields merely hidden by the frontend.

## Skipped, repeated and reordered actions

- Skip the payment step and attempt to confirm a booking.
- Repeat a coupon after applying another valid coupon.
- Replay a used password-reset token.
- Replay confirmation, cancellation, refund and notification requests.
- Reorder payment, confirmation and ticket-issuance requests.
- Attempt to reactivate an expired or cancelled record.
- Repeat the final request after the operation has already succeeded.

## State-transition tests

- Confirm that only documented transitions are accepted.
- Reject unknown source or destination states.
- Reject `PENDING_PAYMENT -> CONFIRMED` without trusted payment evidence.
- Invalidate a ticket after cancellation or refund.
- Prevent a refunded booking from returning to a paid or active state without an approved recovery flow.
- Recheck the authoritative current state immediately before changing it.

## Authorization and ownership

- Use two users and exchange resource identifiers.
- Test read, update, delete, download, export and administrative actions separately.
- Test normal, alternate, legacy, batch and direct-service routes.
- Test each role against the same sensitive action.
- Confirm that denial returns no private data and produces no state change.
- Confirm that private data is filtered before building list and search responses.

## Session and recovery

- Use two browsers and test an old session after logout.
- Repeat after password reset, password change, account disablement and role change.
- Test modified, expired, used and cross-account reset tokens.
- Compare login, registration and recovery responses for existing and non-existing accounts.
- Verify that response status, body, length, redirect and timing do not create a reliable enumeration oracle.

## Concurrency and idempotency

- Send multiple booking requests for the final available place.
- Send duplicate payment and refund requests concurrently.
- Reuse the same idempotency key and confirm that the original result is returned.
- Use different keys for the same business operation and confirm that domain constraints still prevent duplicates.
- Confirm that capacity, balances and state never become negative or contradictory.

## Webhooks and third-party messages

- Send unsigned and incorrectly signed webhooks.
- Modify the signed payload.
- Replay a previously valid event.
- Send an expired or out-of-order event.
- Use the wrong event type, amount, currency, booking or provider reference.
- Confirm that no internal state changes before authenticity, freshness and expected context are verified.

## Failure and timeout behaviour

- Simulate provider timeout before and after the external operation succeeds.
- Retry after a partial failure.
- Confirm that the system does not fail open.
- Confirm that recovery does not duplicate payments, refunds, bookings or notifications.
- Verify that unknown states require review or safe reconciliation rather than automatic success.

## Completed lab-specific checks

### Client-controlled price

- Submit `price=0001` for a high-value product.
- The backend must use the server-side price or reject the request.
- The final checkout must recalculate the authoritative total.

### Coupon workflow

- Apply `NEWCUST5`, then `SIGNUP30`, then `NEWCUST5` again.
- The second use of a single-use coupon must be rejected across the full cart or customer history defined by the promotion.
- Checkout must independently validate the complete set of applied promotions.
