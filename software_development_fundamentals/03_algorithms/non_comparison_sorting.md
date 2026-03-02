# Non-Comparison Sorting Algorithms

## Overview

Non-comparison sorting algorithms break through the O(n log n) lower bound that applies
to all comparison-based sorts by exploiting additional structure in the data -- typically
the fact that elements are integers within a known range or can be decomposed into digits.
Instead of comparing elements pairwise, these algorithms distribute elements into buckets
or count occurrences, enabling linear-time O(n) performance under the right conditions.

The three major non-comparison sorts are counting sort, radix sort, and bucket sort. Each
has specific preconditions: counting sort requires integer keys in a bounded range, radix
sort works on integers or fixed-length strings by processing individual digits, and bucket
sort assumes a uniform distribution of values. When these preconditions are met, these
algorithms can dramatically outperform comparison sorts.

Understanding when to apply non-comparison sorts is a critical skill. They are not
universal replacements for comparison sorts -- they trade generality for speed. An
algorithm that runs in O(n + k) time may be worse than O(n log n) if k is enormous.
This chapter provides the tools to make that judgment.

## Key Concepts

### Why Linear Time is Possible

The O(n log n) lower bound applies only when the sole operation on elements is pairwise
comparison. Non-comparison sorts use different operations:
- **Counting sort:** Uses element values as array indices
- **Radix sort:** Processes individual digits independently
- **Bucket sort:** Maps values to buckets using arithmetic

### The Role of Input Constraints

Linear-time sorting always comes with constraints on the input:

| Algorithm     | Constraint                                    |
|--------------|-----------------------------------------------|
| Counting Sort | Integer keys in range [0, k]                  |
| Radix Sort   | Fixed number of digits d, each in range [0, k] |
| Bucket Sort  | Values uniformly distributed in [0, 1)         |

### Stability Requirement

Both counting sort and the standard radix sort implementation are stable, meaning
equal elements maintain their original relative order. This stability is essential
for radix sort correctness, since it processes digits from least significant to
most significant, relying on stability to preserve the ordering of earlier digits.

## Algorithms and Pseudocode

### 1. Counting Sort

Counting sort counts the occurrences of each value, then uses prefix sums to place
each element directly into its correct position in the output.

```
FUNCTION counting_sort(A: Array of Integer, n: Integer, k: Integer) -> Array of Integer:
    // A contains integers in range [0, k]
    // Step 1: Count occurrences
    SET count = NEW Array[k + 1] initialized to 0
    FOR i = 0 TO n - 1 DO
        SET count[A[i]] = count[A[i]] + 1
    END FOR

    // Step 2: Compute prefix sums (cumulative counts)
    FOR i = 1 TO k DO
        SET count[i] = count[i] + count[i - 1]
    END FOR

    // Step 3: Build output array (traverse backwards for stability)
    SET output = NEW Array[n]
    FOR i = n - 1 DOWNTO 0 DO
        SET output[count[A[i]] - 1] = A[i]
        SET count[A[i]] = count[A[i]] - 1
    END FOR

    RETURN output
END FUNCTION
```

**Step-by-step trace on A = [4, 2, 2, 8, 3, 3, 1], k = 8:**

```
Step 1 - Count occurrences:
  Value:  0  1  2  3  4  5  6  7  8
  Count:  0  1  2  2  1  0  0  0  1

Step 2 - Prefix sums:
  Value:  0  1  2  3  4  5  6  7  8
  Count:  0  1  3  5  6  6  6  6  7

  Meaning: there are 0 elements <= 0, 1 element <= 1, 3 elements <= 2, etc.

Step 3 - Build output (right to left for stability):
  i=6: A[6]=1, count[1]=1, output[0]=1, count[1]->0
  i=5: A[5]=3, count[3]=5, output[4]=3, count[3]->4
  i=4: A[4]=3, count[3]=4, output[3]=3, count[3]->3
  i=3: A[3]=8, count[8]=7, output[6]=8, count[8]->6
  i=2: A[2]=2, count[2]=3, output[2]=2, count[2]->2
  i=1: A[1]=2, count[2]=2, output[1]=2, count[2]->1
  i=0: A[0]=4, count[4]=6, output[5]=4, count[4]->5

  Output: [1, 2, 2, 3, 3, 4, 8]
```

### Counting Sort Prefix Sum Visualization

```
Input Array:  [4, 2, 2, 8, 3, 3, 1]

Count Array (after counting):
Index:    0   1   2   3   4   5   6   7   8
Count:  [ 0 | 1 | 2 | 2 | 1 | 0 | 0 | 0 | 1 ]

Prefix Sum (cumulative):
Index:    0   1   2   3   4   5   6   7   8
Count:  [ 0 | 1 | 3 | 5 | 6 | 6 | 6 | 6 | 7 ]
              ^       ^
              |       |
         1 element  5 elements
         at most 1  at most 3

Output Construction:
Position: [ 0 | 1 | 2 | 3 | 4 | 5 | 6 ]
Output:   [ 1 | 2 | 2 | 3 | 3 | 4 | 8 ]
```

### 2. Radix Sort

Radix sort sorts integers digit by digit, from least significant digit (LSD) to most
significant digit (MSD), using a stable sort (typically counting sort) for each digit.

```
FUNCTION radix_sort(A: Array of Integer, n: Integer):
    SET max_val = FIND_MAXIMUM(A, n)
    SET exp = 1                             // current digit place value

    WHILE max_val / exp > 0 DO
        counting_sort_by_digit(A, n, exp)
        SET exp = exp * 10
    END WHILE
END FUNCTION

FUNCTION counting_sort_by_digit(A: Array of Integer, n: Integer, exp: Integer):
    SET output = NEW Array[n]
    SET count = NEW Array[10] initialized to 0

    // Count occurrences of each digit
    FOR i = 0 TO n - 1 DO
        SET digit = FLOOR(A[i] / exp) MOD 10
        SET count[digit] = count[digit] + 1
    END FOR

    // Prefix sums
    FOR i = 1 TO 9 DO
        SET count[i] = count[i] + count[i - 1]
    END FOR

    // Build output (backwards for stability)
    FOR i = n - 1 DOWNTO 0 DO
        SET digit = FLOOR(A[i] / exp) MOD 10
        SET output[count[digit] - 1] = A[i]
        SET count[digit] = count[digit] - 1
    END FOR

    // Copy back
    FOR i = 0 TO n - 1 DO
        SET A[i] = output[i]
    END FOR
END FUNCTION
```

**Step-by-step trace on A = [170, 45, 75, 90, 802, 24, 2, 66]:**

```
Pass 1 - Sort by ones digit (exp = 1):
  170->0  45->5  75->5  90->0  802->2  24->4  2->2  66->6
  After: [170, 90, 802, 2, 24, 45, 75, 66]

Pass 2 - Sort by tens digit (exp = 10):
  170->7  90->9  802->0  2->0  24->2  45->4  75->7  66->6
  After: [802, 2, 24, 45, 66, 170, 75, 90]

Pass 3 - Sort by hundreds digit (exp = 100):
  802->8  2->0  24->0  45->0  66->0  170->1  75->0  90->0
  After: [2, 24, 45, 66, 75, 90, 170, 802]
```

### Radix Sort Digit Processing Visualization

```
Original:   [170,  45,  75,  90, 802,  24,   2,  66]

=== PASS 1: Sort by ONES digit ===

Digit buckets:
  [0]: 170, 90
  [1]: (empty)
  [2]: 802, 2
  [3]: (empty)
  [4]: 24
  [5]: 45, 75
  [6]: 66
  [7-9]: (empty)

Collect: [170, 90, 802, 2, 24, 45, 75, 66]

=== PASS 2: Sort by TENS digit ===

Digit buckets:
  [0]: 802, 2
  [2]: 24
  [4]: 45
  [6]: 66
  [7]: 170, 75
  [9]: 90

Collect: [802, 2, 24, 45, 66, 170, 75, 90]

=== PASS 3: Sort by HUNDREDS digit ===

Digit buckets:
  [0]: 2, 24, 45, 66, 75, 90
  [1]: 170
  [8]: 802

Collect: [2, 24, 45, 66, 75, 90, 170, 802]  SORTED
```

### 3. Bucket Sort

Bucket sort distributes elements into buckets based on their value, sorts each bucket
individually, then concatenates the results.

```
FUNCTION bucket_sort(A: Array of Float, n: Integer):
    // Assumes A[i] is in range [0, 1)
    SET buckets = NEW Array of Lists[n]

    // Step 1: Distribute elements into buckets
    FOR i = 0 TO n - 1 DO
        SET bucket_index = FLOOR(n * A[i])
        APPEND A[i] TO buckets[bucket_index]
    END FOR

    // Step 2: Sort each bucket (using insertion sort for small buckets)
    FOR i = 0 TO n - 1 DO
        insertion_sort(buckets[i])
    END FOR

    // Step 3: Concatenate all buckets
    SET k = 0
    FOR i = 0 TO n - 1 DO
        FOR EACH element IN buckets[i] DO
            SET A[k] = element
            SET k = k + 1
        END FOR
    END FOR
END FUNCTION
```

**Step-by-step trace on A = [0.78, 0.17, 0.39, 0.26, 0.72, 0.94, 0.21, 0.12, 0.23, 0.68]:**

```
n = 10, so bucket_index = FLOOR(10 * value)

Step 1 - Distribute into 10 buckets:
  Bucket 0: (empty)
  Bucket 1: 0.17, 0.12
  Bucket 2: 0.26, 0.21, 0.23
  Bucket 3: 0.39
  Bucket 4: (empty)
  Bucket 5: (empty)
  Bucket 6: 0.68
  Bucket 7: 0.78, 0.72
  Bucket 8: (empty)
  Bucket 9: 0.94

Step 2 - Sort each bucket:
  Bucket 1: 0.12, 0.17
  Bucket 2: 0.21, 0.23, 0.26
  Bucket 7: 0.72, 0.78

Step 3 - Concatenate:
  [0.12, 0.17, 0.21, 0.23, 0.26, 0.39, 0.68, 0.72, 0.78, 0.94]
```

### Bucket Sort Distribution Visualization

```
Input: [0.78, 0.17, 0.39, 0.26, 0.72, 0.94, 0.21, 0.12, 0.23, 0.68]

Bucket Distribution (10 buckets for range [0.0 - 1.0)):

Bucket  Range          Elements
  0     [0.00, 0.10)   |                             |  (empty)
  1     [0.10, 0.20)   | 0.17  0.12                  |  2 items
  2     [0.20, 0.30)   | 0.26  0.21  0.23            |  3 items
  3     [0.30, 0.40)   | 0.39                        |  1 item
  4     [0.40, 0.50)   |                             |  (empty)
  5     [0.50, 0.60)   |                             |  (empty)
  6     [0.60, 0.70)   | 0.68                        |  1 item
  7     [0.70, 0.80)   | 0.78  0.72                  |  2 items
  8     [0.80, 0.90)   |                             |  (empty)
  9     [0.90, 1.00)   | 0.94                        |  1 item

After sorting each bucket and concatenating:
[0.12, 0.17, 0.21, 0.23, 0.26, 0.39, 0.68, 0.72, 0.78, 0.94]
```

## Complexity Analysis

| Algorithm      | Best    | Average | Worst    | Space    | Stable? |
|---------------|---------|---------|----------|----------|---------|
| Counting Sort | O(n+k)  | O(n+k)  | O(n+k)   | O(n+k)   | Yes     |
| Radix Sort    | O(d(n+k))| O(d(n+k))| O(d(n+k))| O(n+k)  | Yes     |
| Bucket Sort   | O(n+k)  | O(n+k)  | O(n^2)   | O(n+k)   | Yes*    |

Where:
- n = number of elements
- k = range of values (counting sort) or base (radix sort) or number of buckets
- d = number of digits (radix sort)
- *Bucket sort stability depends on the sub-sort used

### When is O(n + k) Actually Better than O(n log n)?

```
Comparison: n = 1,000,000 elements

If k = 100 (small range):
  Counting sort: O(n + k) = 1,000,100     MUCH faster
  Merge sort:    O(n log n) ~ 20,000,000

If k = 1,000,000,000 (huge range):
  Counting sort: O(n + k) = 1,001,000,000  MUCH slower
  Merge sort:    O(n log n) ~ 20,000,000

Rule of thumb: counting sort wins when k = O(n)
```

## Visual Walkthrough

### Counting Sort: The Prefix Sum Trick

```
Why prefix sums give correct positions:

count[v] after prefix sum = number of elements <= v

So if count[3] = 5, element 3 should go at index 4 (the 5th position, 0-indexed).
After placing it, decrement count[3] to 4, so the next 3 goes at index 3.

Visual proof with values [2, 1, 2, 0]:

Counts:    [1, 1, 2, 0]     (one 0, one 1, two 2s)
Prefix:    [1, 2, 4, 4]     (1 elem <= 0, 2 elems <= 1, 4 elems <= 2)

Place A[3]=0: position = count[0]-1 = 0.  count[0]->0
  output: [0, _, _, _]

Place A[2]=2: position = count[2]-1 = 3.  count[2]->3
  output: [0, _, _, 2]

Place A[1]=1: position = count[1]-1 = 1.  count[1]->1
  output: [0, 1, _, 2]

Place A[0]=2: position = count[2]-1 = 2.  count[2]->2
  output: [0, 1, 2, 2]
```

### Radix Sort: Why LSD Order Works

```
Consider sorting: [329, 457, 657, 839, 436, 720, 355]

If we sorted by MSD first, we would need to track sub-groups.
LSD sorting avoids this because stability preserves earlier sorts.

After sorting by ones:  [720, 355, 436, 457, 657, 329, 839]
After sorting by tens:  [720, 329, 436, 839, 355, 457, 657]
After sorting by hundreds: [329, 355, 436, 457, 657, 720, 839]

Key insight: When we sort by hundreds, elements 457 and 436 both have
'4' in hundreds place. Because tens-sort already ordered 436 before 457
(3 < 5), and our sort is STABLE, 436 stays before 457.
```

## When to Use Each Algorithm

### Counting Sort
- Integer keys in a known, bounded range
- Range k is O(n) or smaller
- Stability is required
- Examples: sorting exam scores (0-100), ages (0-150), ASCII characters (0-127)

### Radix Sort
- Fixed-length integers or strings
- Number of digits d is small and constant
- When O(d * (n + k)) is less than O(n log n)
- Examples: sorting phone numbers, IP addresses, fixed-format dates

### Bucket Sort
- Floating-point values uniformly distributed in a known range
- When the distribution is approximately uniform
- Examples: sorting probabilities, normalized scores, hash values

### When NOT to Use Non-Comparison Sorts
- Unknown or unbounded range of values
- Highly skewed (non-uniform) distributions for bucket sort
- When k >> n (counting sort wastes memory)
- General-purpose sorting where input characteristics are unknown
- Negative numbers (requires offset handling)

## Real-World Applications

- **Database indexing:** Radix sort is used for sorting large collections of fixed-width
  keys such as IP addresses and dates in database systems.

- **Suffix array construction:** Radix sort enables linear-time suffix array construction
  algorithms used in text processing and bioinformatics.

- **Graphics rendering:** Radix sort is used in GPU-based sorting for particle systems
  and depth sorting in real-time rendering.

- **Histogram computation:** Counting sort principles underlie histogram construction
  in image processing and statistical analysis.

- **Network packet classification:** Radix-based approaches sort and classify network
  packets by fixed-format header fields.

- **External sorting:** Bucket sort principles are used when distributing data across
  multiple storage devices for parallel sorting.

## Common Pitfalls and Best Practices

1. **Using counting sort with huge ranges.** If k >> n, counting sort wastes memory
   and time. Check that k is proportional to n before choosing counting sort.

2. **Forgetting stability in radix sort.** Radix sort requires a stable sub-sort.
   Using an unstable sort for individual digits produces incorrect results.

3. **Assuming uniform distribution for bucket sort.** If data is heavily skewed, most
   elements end up in a few buckets, degrading to O(n^2) in the worst case.

4. **Ignoring negative numbers.** Standard counting and radix sort assume non-negative
   values. Handle negatives by offsetting or sorting positive and negative separately.

5. **Not considering the constant factors.** Non-comparison sorts have larger constant
   factors than simple comparison sorts. For small n, insertion sort may be faster.

6. **Choosing the wrong radix for radix sort.** A larger base (radix) reduces the
   number of passes d but increases the counting array size k. Balance d and k for
   optimal performance.

## Practice Problems

1. Modify counting sort to handle negative integers in the range [-k, k].

2. Trace radix sort on the array [329, 457, 657, 839, 436, 720, 355] and show the
   array state after sorting by each digit position.

3. Explain why bucket sort degrades to O(n^2) worst case. Describe an input
   distribution that triggers this behavior.

4. Given one million 32-bit integers, compare the expected performance of radix sort
   (base 256, 4 passes) versus merge sort. Which is faster and why?

5. Design a version of radix sort for sorting strings of equal length. Describe how
   to handle the character comparison at each position.

6. Implement bucket sort for integers in the range [0, 999] using 10 buckets of
   width 100 each. Trace on the input [872, 135, 241, 873, 339, 102, 871].

7. Prove that counting sort is stable by examining the backward traversal in step 3.
   What happens if the traversal goes forward instead?
