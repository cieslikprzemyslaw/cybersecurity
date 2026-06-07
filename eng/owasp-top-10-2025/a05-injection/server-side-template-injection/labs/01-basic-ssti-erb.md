# Lab: Basic Server-Side Template Injection - ERB

## Platform

PortSwigger Web Security Academy

Lab:

- https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic

## Status

**Solved**

## Objective

Use the unsafe ERB template construction to delete `morale.txt` from Carlos's home directory.

The target path was provided by the lab description. It was not discovered through filesystem enumeration.

## Vulnerable feature

An out-of-stock product flow caused a message to be displayed on the home page.

The request contained:

```http
GET /?message=Unfortunately%20this%20product%20is%20out%20of%20stock
```

## Controlled input

```text
message
```

I initially thought `productId` was the relevant controlled input because I was reviewing product requests. This was incorrect. `productId` selected a product, while `message` was the value passed into the vulnerable rendering flow.

## Step 1: prove reflection

I changed the value to a unique marker:

```http
GET /?message=SSTI_TEST_123
```

The response contained:

```html
<div>SSTI_TEST_123</div>
```

### Evidence

This proved:

- the parameter was controllable,
- the value was reflected in the response,
- the output location was known.

It did not prove template evaluation.

## Step 2: prove ERB evaluation

The lab description identified ERB. I used a harmless arithmetic expression in ERB output syntax:

```erb
<%= 7*7 %>
```

The response contained:

```html
<div>49</div>
```

### Evidence

The server returned the result of the expression rather than the literal template syntax.

```text
message input
  -> ERB parser/evaluator
  -> calculated result
  -> generated HTML
```

This confirmed SSTI.

## Step 3: perform the lab objective

I did not know Ruby, so I needed to identify a Ruby API for deleting a file.

Ruby provides:

```ruby
File.delete(path)
```

The final authorised-lab expression called `File.delete()` with the path specified by PortSwigger:

```erb
<%= File.delete('/home/carlos/morale.txt') %>
```

URL-encoded in the request, this was sent through the vulnerable `message` parameter.

### Why use `File.delete()`?

- ERB was evaluating Ruby.
- Ruby already provides a direct filesystem API.
- Starting a shell and executing `rm` was unnecessary.
- This clearly demonstrated that SSTI could reach a dangerous server-side API.

## Result

The file was deleted and the lab status changed to solved.

## Impact

High impact was demonstrated because an attacker-controlled template expression could alter the server filesystem within the permissions of the application process.

The evidence did not prove root or unrestricted filesystem access. The process permissions still define the reachable impact.

## Root cause

The application placed the user-controlled `message` value inside ERB template source and then rendered it.

```text
user input became template code
```

The vulnerable design confused data with executable template syntax.

## Secure implementation

- Use a static ERB template controlled by the developer.
- Pass a message only as a variable/data value.
- Do not construct ERB source by concatenating request input.
- Prefer a fixed status identifier such as `out_of_stock` and map it to a server-controlled message.
- Restrict template context and process permissions.
- Do not expose detailed ERB errors in production.

## Regression tests

### Evaluation test

Given a `message` containing ERB syntax, the output must contain literal or encoded text and must not contain the calculated result.

### Side-effect test

Template-looking input must not call Ruby methods or create, modify, or delete files in the test environment.

### Functional test

The valid out-of-stock message must still render correctly after the fix.

## What I misunderstood

- I initially chose `productId` instead of `message`.
- I first treated reflection as if it might already prove SSTI.
- I did not know the file path came directly from the lab description.
- I did not know Ruby's `File.delete()` API.
- I initially described the fix mainly as sanitisation.
- I could not initially define a regression test that distinguished reflection from evaluation.

## Final takeaway

The successful expression was less important than the reasoning sequence:

```text
find controlled input
  -> prove reflection
  -> prove template evaluation
  -> identify the runtime capability needed for the objective
  -> demonstrate impact
  -> explain root cause and secure design
```
