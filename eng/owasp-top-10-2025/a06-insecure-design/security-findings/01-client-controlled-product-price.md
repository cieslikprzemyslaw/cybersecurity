# Client-Controlled Product Price Accepted by Checkout

## Summary

The cart endpoint accepted a product price supplied by the client and used it to calculate the order total.

A customer could modify the `price` field in an add-to-cart request and complete checkout at the manipulated amount.

## Suggested severity

High

The exact severity depends on product values, payment settlement, fraud monitoring and whether affected orders are automatically fulfilled. The authorised lab demonstrated direct financial impact because a high-value product was purchased for a minimal amount.

## Affected component

```text
POST /cart
Parameter: price
Checkout workflow
```

## Vulnerable behaviour

The client submitted:

```text
productId
quantity
price
```

The server treated the submitted `price` as authoritative instead of loading the product price from trusted server-side product data.

Conceptual vulnerable flow:

```text
client selects product
  -> client submits product ID, quantity and price
  -> server trusts submitted price
  -> cart stores manipulated amount
  -> checkout accepts manipulated total
```

## Evidence

A product was added using:

```http
productId=2&redir=PRODUCT&quantity=1&price=0001
```

The cart displayed the product for `$0.01`.

A second test used:

```http
productId=1&redir=PRODUCT&quantity=10&price=0001
```

The normal unit price shown by the application was `$1337.00`, but the cart total became `$0.10`.

Checkout completed successfully.

This confirmed that the modified value affected server-side order processing rather than only the local UI.

## Root cause

A value controlled by an untrusted client was used as an authoritative financial value.

The missing design requirement was that pricing and totals must be calculated from trusted server-side data.

## Why this is Insecure Design

The unsafe result can exist even if the code perfectly implements the request contract.

If the design intentionally accepts and trusts an authoritative client price, a local validation patch does not fix the trust decision. The contract and calculation flow must change.

## Impact

An attacker could potentially:

- purchase products below the intended price,
- create inconsistent order and payment records,
- cause direct revenue loss,
- abuse automated fulfilment,
- combine the issue with quantity or promotion weaknesses.

The lab confirmed a successful purchase at a manipulated amount.

## Security requirement

> The server must calculate product prices and complete order totals using trusted server-side product and promotion data. Client-supplied price values must not influence the authoritative amount.

## Remediation

1. Remove the authoritative `price` field from the client request.
2. Load the current product price by server-selected `productId`.
3. Validate quantity, availability and promotion eligibility server-side.
4. Recalculate the full order total during checkout.
5. Use the same authoritative pricing rules across cart, checkout and fulfilment.
6. Bind the resulting amount to the order and payment operation.
7. Reject inconsistent or stale pricing state safely.
8. Log manipulation attempts for detection, without treating logging as the fix.

## Regression tests

- Submit `price=0001`; the stored and charged amount must remain the correct server-side price.
- Omit the price field; the order must still be priced correctly.
- Modify the price after cart creation; checkout must recalculate the total.
- Modify quantity with zero, negative and excessive values.
- Test every cart, checkout, API-version and batch route.
- Confirm that denied or ignored values cannot change order state.
- Confirm that payment amount matches the final server-calculated order amount.

## Authorised reproduction summary

1. Add a product normally and capture the request.
2. Identify `productId`, `quantity` and `price`.
3. Modify only `price` to a minimal positive value.
4. Confirm that the cart persists the changed amount.
5. Complete checkout.
6. Record the normal price, manipulated total and successful order result.
7. Separate confirmed evidence from assumptions about the backend implementation.

## References

- [OWASP A06:2025 - Insecure Design](https://owasp.org/Top10/2025/A06_2025-Insecure_Design/)
- [PortSwigger Business Logic Vulnerabilities](https://portswigger.net/web-security/logic-flaws)
- [PortSwigger lab: Excessive trust in client-side controls](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls)
