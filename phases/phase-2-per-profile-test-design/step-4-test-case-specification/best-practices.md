# Best Practices — Step 4: Test Case Specification

- Size parameterized data pools using real access-pattern distribution (two-tier weighted or Zipfian), not just uniqueness.
- Explicitly flag every field with a system-enforced uniqueness constraint as a test-environment-preparation concern.
- Specify verification points as the actual success condition, not just "got a response."
- Never let error-handling specifications silently assume automatic retry is safe for a transaction with a real-world side effect.
- Keep the Script Blueprint free of any tool-specific syntax.
- Produce the intermediate Test Data Specification and the final Test Case Specification as genuinely separate documents.
- Cross-check that every correlation point has a corresponding verification point confirming the extraction succeeded.
