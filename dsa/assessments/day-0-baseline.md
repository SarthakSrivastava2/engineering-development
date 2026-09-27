# Day 0 — DSA Baseline Assessment

**Date:** 2026-09-27
**Language:** Go
**Purpose:** Establish a baseline in algorithmic reasoning, complexity analysis, Go implementation fluency, and pattern recognition before systematic DSA training. This was a diagnostic, not a teaching session.

## Diagnostic findings

### Second largest and invariants

The maximum was correctly found with a one-pass scan initialized from the first element: O(n) time and O(1) auxiliary space. The user correctly reasoned that finding the maximum and then scanning again remains O(n) asymptotically.

For a one-pass second-largest solution, the user recognized that two tracked values (`max1` and `max2`) could work but did not independently derive the update rule. The useful invariant is: after processing the first k elements, the two values hold the largest and second-largest values seen so far. When a new value exceeds the largest, the old largest becomes second-largest; otherwise, a value between the two updates second-largest.

**Finding:** Linear scans are familiar; invariant-based reasoning is not yet intuitive.

### Two Sum and hashing

The user identified the nested-loop brute-force approach and correctly recognized its O(n²) time. They initially classified starting the inner loop later as O(n log n); this only removes a constant fraction of comparisons. Total work remains triangular and O(n²).

The complement idea (`target - current`) was present, but the user did not independently reach a hash lookup. The key question is whether the complement has already been seen; a map can provide expected O(1) lookup, yielding expected O(n) time and O(n) space.

**Finding:** Brute force is readily identified. The idea of storing information to make lookup cheaper is emerging, but hashing is not yet an automatic pattern.

### Go range semantics

For a slice loop using `for _, n := range nums`, the user correctly predicted that modifying `n` does not modify the slice because the range variable contains a copy of the element.

**Finding:** Basic Go slice iteration semantics are understood.

### Big-O classification

The user correctly classified a single loop as O(n), a full nested loop as O(n²), and a loop that doubles its counter as O(log n). A shrinking-bound nested loop was classified as O(n log n), but its iteration count is approximately `n + (n-1) + ... + 1`, hence O(n²).

**Recurring finding:** Triangular nested loops were underestimated in both this diagnostic and Two Sum. Count total iterations rather than infer complexity from the loop's changing bound.

### Contains Duplicate

The user first proposed pairwise comparison and correctly identified O(n²) time. In the attempted implementation, the inner loop began at `j := i`, so each element was compared with itself and every non-empty input would return true. The comparison should begin at `i + 1`, and the function needs a final `return false`.

After the Two Sum discussion, the user transferred the hashing idea to this problem: retain previously seen values and check membership before inserting. The algorithmic idea was correct (expected O(n) time, O(n) auxiliary space), while the attempt had Go map syntax/name inconsistencies, no final false return, and stored indices unnecessarily. For membership-only sets, idiomatic Go can use `map[int]struct{}`.

**Positive signal:** The user transferred a recently discussed hashing idea to a new problem. The syntax issues are observations to reinforce, not evidence of a broad Go weakness.

## Overall baseline

Current level: basic DSA familiarity; not yet interview-ready. The user is not starting from zero.

**Strengths observed:**

- Comfortable with basic Go and linear scans.
- Understands common Big-O categories and sequential O(n) work.
- Can construct brute-force approaches.
- Recognized the potential of storing values for faster lookup and transferred that idea between problems.
- Reasons aloud and surfaces uncertainty rather than masking it.

**Weaknesses to revisit:**

- Invariant formulation and update logic.
- Complexity of triangular/shrinking-bound nested loops.
- Deliberate hash-map/set pattern recognition.
- Off-by-one and self-comparison errors.
- Go map initialization, set representation, and syntax consistency in interview solutions.
- Formalizing a promising direction into a data structure, invariant, and correct implementation.
- Limited systematic interview-problem experience.

## Pattern state

Arrays / linear scan: Developing. Hashing: Emerging. Strings, two pointers, sliding window, stacks / queues, binary search, linked lists, trees, heaps, graphs, intervals, recursion / backtracking, and dynamic programming: Untested.

## Relevant Go baseline

Basic Go is adequate to begin DSA training. Slice iteration and range-copy semantics were handled correctly. Map initialization, idiomatic set usage, minor syntax/name consistency, and careful boundary checks should be reinforced without overstating them as major Go weaknesses.

## Next step

Begin Day 1 with arrays and hashing fundamentals. Emphasize invariants, the question “what information would make the next operation cheap?”, total-iteration counting, and boundary checks. Keep an attempt-first format: ask for the user's approach before code, allow productive struggle, and review reasoning separately from implementation. Do not optimize for problem count.
