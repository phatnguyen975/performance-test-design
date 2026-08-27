# Resource: Test Data Types Reference

Reference categories of test data typically required for a performance test, useful when reviewing Data Parameterization Specification for completeness.

## User/Identity Data

Accounts, credentials, session-eligible identities — usually the largest-volume parameterization need.

## Transactional Input Data

Data a user provides while performing a transaction — search terms, cart contents, addresses, form field values. Needs realistic diversity, not just uniqueness.

## Reference/Lookup Data

Data the system already holds that a transaction references (product catalog entries, valid coupon codes, valid shipping zones). Test environment must be seeded with a realistic volume/distribution of this data.

## System/Environment Configuration Data

Not typically parameterized per-VU, but must be documented as a precondition: feature flags, environment-specific endpoints, sandbox/test-mode credentials for third-party integrations.

## State-Dependent Data

Data whose validity depends on the system's current state and can be consumed/invalidated by the test itself — single-use coupon codes, limited-inventory SKUs, one-time-use vouchers. Needs explicit reset/reseed planning between test runs.

## Why This Belongs Here

When completing Data Parameterization Specification, use this list as a completeness check — walk each category and confirm this profile's transactions have been checked against it.
