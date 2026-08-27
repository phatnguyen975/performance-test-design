# Technique: Correlation Mapping Specification

**ISTQB CT-PT reference:** 4.2.7 — dynamic data aspect, specification scope only.

## What It Is

Correlation is capturing a dynamic value returned by one step's response (a session token, an order ID, a CSRF token, a server-computed price) and feeding it into a subsequent step's request. Without correlation, a multi-step transaction with session state or server-generated identifiers cannot be replayed correctly.

This technique **specifies** every point in the flow where this extraction-and-reuse must happen — not the actual extraction code.

## When to Use

For every multi-step transaction (from Step 1) involving session state, server-generated identifiers, or any value computed server-side that a later step depends on.

## When NOT to Use

Not needed for single-step, stateless transactions. Not needed for values the client already knows in advance (those are Data Parameterization Specification's concern).

## How to Apply

1. Walk through each multi-step transaction's flow (Step 1's Transaction Identification) step by step.
2. At each step, ask: does this step's request need any value only made available by an earlier step's response?
3. For each such value, specify: source step/field, destination step(s)/field, and lifetime (session-long vs. single-use).
4. Flag any correlation point tied to a short-expiry value (e.g., a time-limited CSRF token) — affects how tightly the Script Blueprint must chain extraction and reuse.

## Output

A correlation map per transaction.

## Example (Complete Checkout transaction)

| Value                      | Source Step/Field                             | Destination Step(s)/Field                         | Lifetime                                                        |
| -------------------------- | --------------------------------------------- | ------------------------------------------------- | --------------------------------------------------------------- |
| `session_token`            | Login response → `session.token`              | Every subsequent request → `Authorization` header | Session duration                                                |
| `cart_id`                  | Add to Cart response → `cart.id`              | Apply Coupon, Confirm Payment → body `cart_id`    | Until Submit Order completes                                    |
| `csrf_token`               | Checkout page load → response header          | Confirm Payment → header `X-CSRF-Token`           | **Single-use — re-extract on retry, don't reuse a stale token** |
| `payment_authorization_id` | Confirm Payment response → `authorization.id` | Submit Order → body `payment_authorization_id`    | Single-use                                                      |
| `order_id`                 | Submit Order response → `order.id`            | Fulfillment Dispatch polling → path parameter     | Up to 60s polling window (Step 1's transaction boundary)        |
