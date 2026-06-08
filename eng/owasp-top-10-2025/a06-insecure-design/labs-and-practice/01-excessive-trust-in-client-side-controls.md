# Lab: Excessive Trust in Client-Side Controls

## Source

[PortSwigger lab](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls)

## Intended business rule

A customer may purchase a product only for the price defined by the store and only when sufficient credit is available.

## Initial observation

The add-to-cart request contained:

```text
productId
quantity
price
```

`quantity` is legitimately selected by the customer, although the backend must still validate its allowed range.

The important question was whether `price` was only display data or an authoritative value trusted by the server.

## Hypothesis

The backend may use the client-supplied `price` instead of loading the product price from trusted server-side data.

## Controlled test

Only the price was changed:

```http
POST /cart

productId=2&redir=PRODUCT&quantity=1&price=0001
```

The server returned a redirect and the cart displayed the product for `$0.01`.

This proved more than a local DOM change because the modified value was stored in server-side cart state.

## Impact confirmation

A second request added ten units of a product whose normal unit price was `$1337.00`:

```http
POST /cart

productId=1&redir=PRODUCT&quantity=10&price=0001
```

The cart total became `$0.10`, and checkout completed successfully.

## Facts

- The request contained a client-controlled price.
- The server accepted `price=0001`.
- The cart used the modified price.
- The final checkout accepted the manipulated total.

## Unsafe assumption

The design assumed that the browser would submit the store's correct price.

## Root cause

A client-controlled value was used as an authoritative financial value.

The issue was not merely that Burp could edit a request. The server delegated a business-critical calculation to an untrusted client.

## Why this maps to Insecure Design

Even technically correct code would remain unsafe if the intended design says:

```text
receive product ID, quantity and price
  -> trust all three values
  -> create the cart total
```

The required control belongs in the design:

```text
receive product ID and quantity
  -> load product price from trusted server-side data
  -> validate quantity
  -> calculate and re-calculate the total server-side
```

## Impact

An attacker could purchase products for a manipulated amount, causing direct financial loss and inconsistent order records.

## Security requirement

> The server must calculate product prices and order totals using trusted server-side product data. Any price supplied by the client must be ignored or rejected.

## Remediation

- Remove the authoritative price from the client contract.
- Load price and promotion data using the server-selected `productId`.
- Validate allowed quantity and product availability server-side.
- Recalculate the complete order at checkout.
- Use one authoritative pricing service or rule set across cart and checkout.
- Log rejected price-manipulation attempts as detection, not as the primary fix.

## Regression tests

- Submit a valid product and quantity with `price=0001`; the server must use the correct stored price or reject the field.
- Remove the price field; the correct price must still be calculated.
- Modify the price after adding the product but before checkout; checkout must recalculate the total.
- Test zero, negative and excessively large quantities.
- Confirm that no alternate cart or checkout endpoint accepts a client-defined amount.

## Learning process

I initially focused on `quantity` because it was clearly user-controlled.

The stronger candidate was `price`, because the client should not make the final pricing decision. The decisive evidence was not the redirect response itself but the persisted cart value and successful checkout.
