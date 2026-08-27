# Technique: Data Parameterization Specification

**ISTQB CT-PT reference:** 4.2.7 — Implementing Performance Test Scripts (data aspect, specification scope only).

## What It Is

Parameterization means replacing hard-coded values with variables drawn from a dataset, so each virtual user/iteration uses different, realistic data. This technique **specifies** what needs to be parameterized and how — it does not build the actual data files or write script code.

Insufficient parameterization is one of the most common causes of a misleading performance test: reusing a small, fixed set of values causes application/database caches to warm up after the first few iterations, making the system appear far faster than it would with real, diverse production traffic.

## When to Use

For every field in every transaction (from Step 1) that would plausibly differ per real user/session.

## When NOT to Use

Not needed for fields genuinely constant across all real usage (e.g., an API version header).

## How to Apply

1. Walk through each transaction's steps and list every input field the request requires.
2. For each field, decide: constant (no parameterization) or varies (needs parameterization).
3. For each field needing parameterization, specify: data type/format, required volume (minimum unique values — see Data Diversity Rules for calculation), source (desensitized production extract, synthetic set matching production's distribution, or test-environment-seeded), and reuse policy (cycle through pool vs. single-use-per-run).
4. Flag any field with a system-enforced uniqueness constraint (e.g., a coupon code redeemable once) — these need either a much larger unique pool or a data-reset step between test runs, a test-environment-preparation concern, not just a scripting detail.

## Output

A field-by-field parameterization table per transaction.

## Example (Confirm Payment transaction)

| Field                             | Type/Format                                 | Required Volume                                                                                                         | Source                                                  | Reuse Policy                               | Uniqueness Constraint                                        |
| --------------------------------- | ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| `customer_id`                     | Existing registered user ID                 | ≥ 550 unique (matching final VU count from Step 3)                                                                      | Desensitized production user table extract              | Cycle through pool                         | No                                                           |
| `cart_total`                      | Decimal, realistic order-value distribution | Derived dynamically per iteration                                                                                       | Generated within the scenario (see Correlation Mapping) | N/A — dynamic                              | No                                                           |
| `payment_method_token`            | Gateway test-mode token                     | ≥ 550 unique                                                                                                            | Payment gateway sandbox token issuance                  | Cycle through pool                         | No                                                           |
| `coupon_code` (Apply Coupon only) | Alphanumeric, system format                 | ≥ 700 unique, single-use codes (sized for the full test run's expected Apply Coupon iteration count, not just VU count) | Pre-generated test-mode batch, seeded before run        | **Single-use — do not reuse within a run** | **Yes — flag for test data reset/reseed before repeat runs** |
