# Technique: Script Blueprint Specification

**ISTQB CT-PT reference:** 4.2.6 — Basic Structure of a Performance Test Script (structural/logical description only — no tool syntax or code).

## What It Is

ISTQB CT-PT describes a performance test script's structure as generally consisting of an initialization section, one or more main action sections representing the transactions under test, and a cleanup/termination section — plus cross-cutting concerns like parameterization, correlation, response verification, and error handling.

This technique specifies that **logical structure** for the profile's scenario in plain, tool-agnostic terms.

## When to Use

Once Data Parameterization, Correlation Mapping, and Data Diversity are all specified — this comes last among Step 4's techniques since it references all three.

## When NOT to Use

Never include actual tool syntax, code, or a specific tool's configuration format here — the moment this looks like a JMeter test plan XML or a k6 script, it has crossed out of this skill's scope.

## How to Apply

Specify each section for the scenario, in plain structural terms:

1. **Initialization** — what happens once per virtual user before its first iteration.
2. **Main Flow** — the ordered sequence of transactions (from Step 1) the VU executes per iteration, referencing the think time (Step 3), parameterized fields (Data Parameterization), and correlation points (Correlation Mapping) at each step.
3. **Verification Points** — for each step, the specific response condition constituting success (status code AND relevant field checks where applicable) — not just "got a response."
4. **Error Handling** — what the script should do on a verification failure, specified as intent (abort iteration and log vs. retry a bounded number of times), not tool-specific syntax. Explicitly address whether retry is safe for any transaction with a real-world side effect.
5. **Cleanup/Termination** — anything that must happen at the end of a VU's run, only if relevant.

## Output

A structured blueprint document with the five sections filled in.

## Example (Complete Checkout scenario, condensed)

**Initialization:** Authenticate as the assigned `customer_id`; capture `session_token`; abort iteration (log as setup failure) if auth fails.

**Main Flow** (each step separated by Step 3's think time): Browse Catalog → Search → Add to Cart (captures `cart_id`) → [conditional, per Step 1's mix] Apply Coupon → Confirm Payment (captures `payment_authorization_id`) → Submit Order (captures `order_id`) → Fulfillment Dispatch polling.

**Verification Points:** Confirm Payment: HTTP 200 **and** `authorization.status == "approved"` — a 200 status alone is insufficient, since the gateway can return 200 with a declined status. Submit Order: HTTP 201 **and** `order.id` present.

**Error Handling:** Any verification failure aborts the remainder of that VU's current iteration, logs the failure with transaction name and step. **No automatic retry on Confirm Payment or Submit Order** — a retried payment against a real/sandboxed gateway risks a duplicate charge.

**Cleanup:** None beyond the natural end of iteration.
