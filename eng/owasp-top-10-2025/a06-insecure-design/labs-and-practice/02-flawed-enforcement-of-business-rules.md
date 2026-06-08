# Lab: Flawed Enforcement of Business Rules

## Source

[PortSwigger lab](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-flawed-enforcement-of-business-rules)

## Intended business rule

Promotional codes must follow their defined usage and combination policy.

A code intended for a new customer or single use must not become reusable simply because another valid code was applied between attempts.

## Observed promotions

The application exposed two legitimate codes:

```text
NEWCUST5
SIGNUP30
```

`NEWCUST5` reduced the order by `$5.00`.

`SIGNUP30` reduced the price by 30%.

## First assumption

The backend might correctly reject a coupon if the same code had already been used.

Direct duplicate use appeared to be restricted, but this did not prove that the full coupon history was validated.

## Controlled sequence

The following sequence was accepted:

```text
NEWCUST5
SIGNUP30
NEWCUST5
SIGNUP30
NEWCUST5
SIGNUP30
NEWCUST5
SIGNUP30
```

The order total for a `$1337.00` product reached `$0.00`.

Checkout then completed successfully.

## Facts

- Both coupon values were legitimate.
- The application accepted both coupon types in the same cart.
- Alternating the codes allowed the same code to be applied repeatedly.
- The complete order was accepted at `$0.00`.

## Inference

The behaviour strongly suggests that duplicate detection considered only the most recently applied coupon or otherwise failed to validate the complete promotion state.

The exact implementation cannot be confirmed without source code.

## Unsafe assumption

The workflow assumed that preventing two identical codes directly next to each other was sufficient to enforce the promotion policy.

## Root cause

The application did not enforce coupon usage rules across the complete authoritative cart, order or customer history required by the promotion.

The problem was not the coupon format. Each input was valid.

## Why input validation would not fix it

Input validation can confirm that:

- the code exists,
- the syntax is valid,
- the code is active,
- the code has not expired.

It cannot fix a state model that allows this sequence:

```text
coupon A -> coupon B -> coupon A
```

The backend must validate history, combination rules and usage scope.

## Security requirement

> The server must enforce each coupon's usage and combination policy across the complete authoritative cart, order and customer history defined by the promotion, including a final validation at checkout.

## Remediation

- Define whether each coupon is limited per account, cart, order or promotion period.
- Store all applied promotions as authoritative server-side state.
- Reject a code already used within its defined scope.
- Enforce allowed and forbidden coupon combinations.
- Recalculate discounts from server-side promotion rules.
- Revalidate the full promotion set during checkout.
- Make coupon application and checkout concurrency-safe.

## Regression tests

- Apply `NEWCUST5`, then `SIGNUP30`, then `NEWCUST5` again; the second `NEWCUST5` must be rejected.
- Attempt direct duplicates and alternating duplicates.
- Repeat the test through every cart and checkout route.
- Re-submit the final coupon request.
- Submit checkout with a manipulated or stale promotion list.
- Test concurrent coupon application.
- Verify that denied requests do not change the cart total.

## Learning process

I first recognised that a second legal coupon might be important.

The key improvement was moving from guessing codes to testing a defined business rule. The vulnerability appeared through the sequence and state history, not through a malformed payload.
