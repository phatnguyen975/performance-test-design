# Output Quality Checklist — Step 4: Test Case Specification (AI Gate)

- [ ] Every field in every transaction is classified as constant or parameterized, with none silently skipped.
- [ ] Every parameterized field specifies data type/format, required volume, source, and reuse policy.
- [ ] Every field with a system-enforced uniqueness constraint is explicitly flagged with its test-environment reset/reseed implication.
- [ ] Every multi-step transaction has a complete correlation map with no missing extraction points.
- [ ] Every correlation point states its lifetime.
- [ ] Every data-volume/cache-sensitive parameterized field has an explicit diversity specification (pool size and distribution shape), with reasoning.
- [ ] The diversity specification uses production's real access distribution where available, rather than defaulting to uniform without checking.
- [ ] The Script Blueprint includes all five sections; genuinely non-applicable sections are stated as such, not silently omitted.
- [ ] Every verification point specifies the actual success condition, not just "received a response."
- [ ] Every correlation point has a corresponding verification that its extraction succeeded.
- [ ] Error handling explicitly addresses retry safety for each transaction with a real-world side effect.
- [ ] No tool-specific syntax, code, or configuration format appears anywhere.
- [ ] The intermediate Test Data Specification and final Test Case Specification exist as two distinct documents; the final one is condensed, not a copy.

If any item fails, fix it here before this profile returns to Human Review Gate 2.
