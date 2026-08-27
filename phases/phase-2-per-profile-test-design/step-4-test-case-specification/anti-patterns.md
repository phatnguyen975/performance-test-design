# Anti-Patterns — Step 4: Test Case Specification

- Reusing a small, fixed data pool across a large virtual-user population.
- Parameterizing uniformly when production access is long-tailed/Zipfian.
- Treating "got an HTTP 200" as sufficient verification for a transaction where the response body itself can indicate failure.
- Specifying automatic retry for a transaction with a real-world side effect without considering duplication risk.
- Writing tool-specific syntax into the Script Blueprint.
- Collapsing the intermediate Test Data Specification and the final Test Case Specification into one document.
- Specifying a correlation point without a corresponding verification that the extraction succeeded.
- Skipping Data Diversity Rules for a field just because it was already covered in Data Parameterization Specification — uniqueness and diversity are different concerns.
