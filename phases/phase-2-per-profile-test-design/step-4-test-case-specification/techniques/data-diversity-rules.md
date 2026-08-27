# Technique: Data Diversity Rules

**Grounding:** ISTQB CT-PT 4.2.7 (data realism aspect, specification scope only), extended with the Zipfian/power-law distribution model commonly used in web-caching research to represent real-world content-popularity skew, applied here to test-data pool composition.

## What It Is

Beyond simply parameterizing a field, the _size and composition_ of the parameterized dataset materially affects test validity. A dataset that's too small causes artificial cache warm-up (the system looks faster than it really is); a dataset that doesn't reflect production's real distribution causes the test to miss performance characteristics tied to data shape.

## When to Use

For every parameterized field (from Data Parameterization Specification) that interacts with a caching layer, an index, or any data-volume-sensitive code path.

## When NOT to Use

Not needed for fields with no caching or data-volume sensitivity.

## How to Apply

1. **Identify cache/index sensitivity** for each parameterized field.
2. **Calculate minimum pool size:** large enough that, across the full planned test duration, no single value receives unrealistic cache-hit treatment compared to production's actual access pattern. Starting heuristic: size the pool to at least match the number of unique values production analytics show being accessed within the equivalent real time window.
3. **Model real-world popularity skew using a Zipfian distribution where production data shows it exists.** Web content access (and by extension, product catalog views, search terms, and similar) very commonly follows a power-law/Zipfian pattern: the k-th most popular item receives roughly `1/k^s` of the traffic of the most popular item, for some skew parameter `s` (commonly close to 1 for web content popularity). Rather than drawing parameterized values uniformly at random from the full pool, weight the draw so the most popular items receive disproportionately more selections — matching how a real cache actually experiences hit/miss behavior. A uniform draw across a long-tail-distributed pool produces a materially different, and misleadingly low, cache-hit rate compared to what production experiences.
4. **Where the true skew parameter isn't known precisely**, a simplified two-tier approximation is an acceptable, clearly-labeled substitute: split the pool into a "hot" sub-pool (e.g., the top ~1–2% of items by known/estimated popularity) and a "long-tail" sub-pool (the rest), and weight draws between them according to whatever concentration production analytics actually show (e.g., "top 500 SKUs = 60% of views"). State explicitly that this is an approximation of the full Zipfian curve, not the curve itself, when the precise skew parameter isn't available.
5. **Document the chosen pool size and distribution shape (uniform, two-tier weighted, or full Zipfian) per field, with reasoning.**

## Output

A diversity specification per data-volume-sensitive field: required pool size, distribution shape and parameters, and reasoning.

## Example

**Product ID field (Browse Catalog, Search, Add to Cart):**

- **Production catalog:** 45,000 active SKUs. Access analytics show the top 500 SKUs account for ~60% of all product-detail views (a two-tier approximation of the underlying Zipfian pattern; the precise skew parameter wasn't available from the analytics platform used).
- **Pool size:** minimum 5,000 unique product IDs.
- **Distribution:** two-tier weighted — 60% of generated requests draw from a 500-ID "hot" sub-pool, 40% from the remaining ~4,500 IDs, drawn uniformly within that group.
- **Reasoning:** a uniform draw across all 5,000 IDs would produce a materially different, lower cache-hit rate at the CDN/application-cache layer than production actually experiences, distorting response-time results for Browse Catalog and Search.

**Coupon code field (Apply Coupon):** No caching/index sensitivity beyond the single-use redemption constraint already flagged in Data Parameterization Specification — pool _size_ here is driven purely by the single-use constraint, not cache-hit-rate concerns. State this explicitly rather than silently omitting a diversity entry.
