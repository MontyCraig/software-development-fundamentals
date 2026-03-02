# Searching Algorithms

## Overview

Searching is one of the most fundamental operations in computer science. Given a
collection of elements, a search algorithm determines whether a target value exists
within the collection and, if so, where it is located. The efficiency of search
operations profoundly impacts overall system performance since searching is a
building block for sorting, indexing, data retrieval, and countless other operations.

The simplest approach, linear search, examines every element and works on any collection.
When the data is sorted, however, far more efficient methods become available. Binary
search eliminates half the remaining candidates with each comparison. Interpolation
search, jump search, and exponential search offer alternative strategies that outperform
binary search under specific conditions such as uniformly distributed data or unbounded
collection sizes.

Choosing the right search algorithm depends on the data structure, whether the data is
sorted, the distribution of values, and the cost of element access. This chapter covers
five major searching algorithms, their preconditions, and the tradeoffs between them.

## Key Concepts

### Preconditions

| Algorithm          | Requires Sorted? | Requires Random Access? | Distribution Assumption?  |
|-------------------|-------------------|-------------------------|---------------------------|
| Linear Search     | No                | No                      | None                      |
| Binary Search     | Yes               | Yes                     | None                      |
| Interpolation     | Yes               | Yes                     | Uniform distribution      |
| Jump Search       | Yes               | Yes                     | None                      |
| Exponential Search| Yes               | Yes                     | None                      |

### Search Space Reduction

The core idea behind efficient searching is reducing the search space -- the set of
candidate positions where the target might exist -- as quickly as possible.

```
Search space reduction strategies:

Linear:        Eliminate 1 element per step
               [? ? ? ? ? ? ? ? ? ?]  ->  [? ? ? ? ? ? ? ? ?]  -> ...
                ^                          ^

Binary:        Eliminate half per step
               [? ? ? ? ? ? ? ? ? ?]  ->  [? ? ? ? ?]  ->  [? ?]  -> [?]
                        ^                      ^             ^

Interpolation: Eliminate proportional to value proximity
               [? ? ? ? ? ? ? ? ? ?]  ->  [? ?]  ->  [?]
                            ^                ^
```

## Algorithms and Pseudocode

### 1. Linear Search

Examine every element from beginning to end until the target is found or the
collection is exhausted.

```
FUNCTION linear_search(A: Array of Comparable, n: Integer, target: Comparable) -> Integer:
    FOR i = 0 TO n - 1 DO
        IF A[i] = target THEN
            RETURN i
        END IF
    END FOR
    RETURN -1                               // not found
END FUNCTION
```

**Step-by-step trace on A = [4, 7, 2, 9, 1, 5, 3], target = 5:**

```
i=0: A[0]=4, 4 != 5 -> continue
i=1: A[1]=7, 7 != 5 -> continue
i=2: A[2]=2, 2 != 5 -> continue
i=3: A[3]=9, 9 != 5 -> continue
i=4: A[4]=1, 1 != 5 -> continue
i=5: A[5]=5, 5 == 5 -> RETURN 5

Found at index 5 after 6 comparisons.
```

### Sentinel Linear Search (Optimization)

```
FUNCTION sentinel_search(A: Array of Comparable, n: Integer, target: Comparable) -> Integer:
    SET last = A[n - 1]
    SET A[n - 1] = target                   // place sentinel
    SET i = 0
    WHILE A[i] != target DO
        SET i = i + 1
    END WHILE
    SET A[n - 1] = last                     // restore original
    IF i < n - 1 OR A[n - 1] = target THEN
        RETURN i
    END IF
    RETURN -1
END FUNCTION

// Eliminates the bounds check (i < n) from the inner loop
```

### 2. Binary Search

Repeatedly divide the sorted search space in half by comparing the target to the
middle element.

```
FUNCTION binary_search(A: Sorted Array, n: Integer, target: Comparable) -> Integer:
    SET low = 0
    SET high = n - 1
    WHILE low <= high DO
        SET mid = low + FLOOR((high - low) / 2)    // avoids overflow
        IF A[mid] = target THEN
            RETURN mid
        ELSE IF A[mid] < target THEN
            SET low = mid + 1
        ELSE
            SET high = mid - 1
        END IF
    END WHILE
    RETURN -1
END FUNCTION
```

**Step-by-step trace on A = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91], target = 23:**

```
Step 1: low=0, high=9, mid=4
        A[4]=16 < 23 -> low=5
        Search space: [23, 38, 56, 72, 91]

        [  2,  5,  8, 12, 16, 23, 38, 56, 72, 91]
                               X   ^-- search here -->

Step 2: low=5, high=9, mid=7
        A[7]=56 > 23 -> high=6
        Search space: [23, 38]

        [  2,  5,  8, 12, 16, 23, 38, 56, 72, 91]
                               <-here->  X

Step 3: low=5, high=6, mid=5
        A[5]=23 = 23 -> RETURN 5

Found at index 5 after 3 comparisons (vs 6 for linear search).
```

### Binary Search Visualization

```
A = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]    Target = 23

Iteration 1:
  [  2 |  5 |  8 | 12 | 16 | 23 | 38 | 56 | 72 | 91 ]
   lo                   mid                         hi
   0                     4                           9
   A[4]=16 < 23, move low to mid+1

Iteration 2:
  [  2 |  5 |  8 | 12 | 16 | 23 | 38 | 56 | 72 | 91 ]
                               lo        mid       hi
                               5          7         9
   A[7]=56 > 23, move high to mid-1

Iteration 3:
  [  2 |  5 |  8 | 12 | 16 | 23 | 38 | 56 | 72 | 91 ]
                               lo   hi
                               mid
                               5    6
   A[5]=23 = 23, FOUND at index 5
```

### 3. Interpolation Search

Instead of always checking the middle, estimate the target position based on its
value relative to the endpoints. Works best on uniformly distributed data.

```
FUNCTION interpolation_search(A: Sorted Array, n: Integer, target: Comparable) -> Integer:
    SET low = 0
    SET high = n - 1
    WHILE low <= high AND target >= A[low] AND target <= A[high] DO
        IF low = high THEN
            IF A[low] = target THEN
                RETURN low
            END IF
            RETURN -1
        END IF
        // Estimate position using linear interpolation
        SET pos = low + FLOOR(
            (target - A[low]) * (high - low) / (A[high] - A[low])
        )
        IF A[pos] = target THEN
            RETURN pos
        ELSE IF A[pos] < target THEN
            SET low = pos + 1
        ELSE
            SET high = pos - 1
        END IF
    END WHILE
    RETURN -1
END FUNCTION
```

**Step-by-step trace on A = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100], target = 70:**

```
Step 1: low=0, high=9
        pos = 0 + (70-10)*(9-0)/(100-10) = 0 + 60*9/90 = 6
        A[6]=70 = 70 -> RETURN 6

Found in 1 comparison! (Binary search would need 3-4.)
With uniformly distributed data, interpolation search jumps close to the answer.
```

### 4. Jump Search

Jump ahead by a fixed step size, then perform linear search in the block where the
target might exist. Balances between linear and binary search.

```
FUNCTION jump_search(A: Sorted Array, n: Integer, target: Comparable) -> Integer:
    SET step = FLOOR(SQRT(n))
    SET prev = 0

    // Jump ahead until we pass the target or reach the end
    WHILE A[MIN(step, n) - 1] < target DO
        SET prev = step
        SET step = step + FLOOR(SQRT(n))
        IF prev >= n THEN
            RETURN -1
        END IF
    END WHILE

    // Linear search within the block [prev, min(step, n))
    WHILE A[prev] < target DO
        SET prev = prev + 1
        IF prev = MIN(step, n) THEN
            RETURN -1
        END IF
    END WHILE

    IF A[prev] = target THEN
        RETURN prev
    END IF
    RETURN -1
END FUNCTION
```

**Step-by-step trace on A = [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89], target = 55:**

```
n=12, step = FLOOR(SQRT(12)) = 3

Jump phase:
  Check A[2]=1 < 55 -> jump. prev=3
  Check A[5]=5 < 55 -> jump. prev=6
  Check A[8]=21 < 55 -> jump. prev=9
  Check A[11]=89 >= 55 -> stop jumping

Linear phase (search from index 9 to 11):
  A[9]=34 < 55 -> next
  A[10]=55 = 55 -> RETURN 10

Found at index 10: 4 jumps + 2 linear comparisons = 6 total.
```

### 5. Exponential Search

Double the search range exponentially until the target is bracketed, then use binary
search within that range. Ideal when the target is near the beginning or the array
size is unknown.

```
FUNCTION exponential_search(A: Sorted Array, n: Integer, target: Comparable) -> Integer:
    IF A[0] = target THEN
        RETURN 0
    END IF

    // Find range by doubling
    SET bound = 1
    WHILE bound < n AND A[bound] <= target DO
        SET bound = bound * 2
    END WHILE

    // Binary search within [bound/2, min(bound, n-1)]
    SET low = FLOOR(bound / 2)
    SET high = MIN(bound, n - 1)
    RETURN binary_search_range(A, low, high, target)
END FUNCTION

FUNCTION binary_search_range(A: Array, low: Integer, high: Integer, target: Comparable) -> Integer:
    WHILE low <= high DO
        SET mid = low + FLOOR((high - low) / 2)
        IF A[mid] = target THEN
            RETURN mid
        ELSE IF A[mid] < target THEN
            SET low = mid + 1
        ELSE
            SET high = mid - 1
        END IF
    END WHILE
    RETURN -1
END FUNCTION
```

**Step-by-step trace on A = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, ...100], target = 7:**

```
A[0]=1 != 7

Exponential range-finding:
  bound=1:  A[1]=2 <= 7   -> bound=2
  bound=2:  A[2]=3 <= 7   -> bound=4
  bound=4:  A[4]=5 <= 7   -> bound=8
  bound=8:  A[8]=9 > 7    -> stop

Binary search in range [4, 8]:
  mid=6: A[6]=7 = 7 -> RETURN 6

Found at index 6.
Range-finding: 4 steps. Binary search: 1 step. Total: 5 comparisons.
```

## Visual Walkthrough

### Search Space Reduction Comparison (n = 16, target at index 11)

```
                    Linear Search
Step:  1  2  3  4  5  6  7  8  9 10 11 12
       X  X  X  X  X  X  X  X  X  X  X  !
       Checks every element: 12 comparisons

                    Jump Search (step = 4)
Step:  1        2        3        4  5  6
       X--------X--------X--------X  X  !
       Jump     Jump     Jump     Linear search in block
       4 jumps + 2 linear = 6 comparisons

                    Binary Search
Step:  1              2        3     4
       --------X------         --X--
                      ----X---     X!
       4 comparisons (halving each time)

                    Exponential Search
Step:  1  2     3           4     5
       X  X-----X-----------X
                      binary search in [4,8]
       4 range + 1 binary = 5 comparisons

Legend: X = comparison point, ! = found, - = skipped elements
```

### Binary Search Decision Tree (n = 7)

```
Sorted array: [1, 3, 5, 7, 9, 11, 13]

                        [7]          compare with mid
                       / \
                      /   \
                   [3]     [11]      compare with mid
                   / \     / \
                  /   \   /   \
                [1]  [5] [9] [13]    compare with mid

Maximum comparisons = tree height = CEIL(log2(8)) = 3

Any element can be found in at most 3 comparisons.
```

## Complexity Analysis

| Algorithm          | Best   | Average     | Worst      | Space |
|-------------------|--------|-------------|------------|-------|
| Linear Search     | O(1)   | O(n)        | O(n)       | O(1)  |
| Sentinel Search   | O(1)   | O(n)        | O(n)       | O(1)  |
| Binary Search     | O(1)   | O(log n)    | O(log n)   | O(1)  |
| Interpolation     | O(1)   | O(log log n)| O(n)       | O(1)  |
| Jump Search       | O(1)   | O(sqrt(n))  | O(sqrt(n)) | O(1)  |
| Exponential Search| O(1)   | O(log n)    | O(log n)   | O(1)  |

### Comparison at Different Scales

```
n = 1,000,000 elements:

Linear:        1,000,000 comparisons (worst)
Jump:          1,000 comparisons (sqrt of n)
Binary:        20 comparisons (log2 of n)
Interpolation: ~4 comparisons (log log n, uniform data)

n = 1,000,000,000 elements:

Linear:        1,000,000,000 comparisons
Jump:          31,623 comparisons
Binary:        30 comparisons
Interpolation: ~5 comparisons (uniform data)
```

## When to Use Each Algorithm

- **Linear Search:** Unsorted data, linked lists, small collections, or when you
  need to find all occurrences (not just the first).

- **Binary Search:** The go-to algorithm for sorted arrays. Reliable O(log n)
  performance regardless of data distribution.

- **Interpolation Search:** Sorted data with approximately uniform distribution.
  Achieves O(log log n) on average but degrades to O(n) on skewed distributions.

- **Jump Search:** Sorted data where backward traversal is expensive (such as
  linked lists with forward-only jumps). Simpler than binary search to implement
  for sequential-access structures.

- **Exponential Search:** When the array size is unknown or unbounded, or when
  the target is likely near the beginning. Also useful for unbounded binary search
  on infinite sequences.

## Real-World Applications

- **Database queries:** Binary search on sorted indices enables O(log n) record
  lookup in B-tree based databases.

- **Autocomplete systems:** Binary search or interpolation search on sorted
  dictionary arrays for prefix matching.

- **Version control:** Binary search (bisect) to find the commit that introduced
  a bug by testing midpoint commits.

- **Network routing:** Binary search on routing tables sorted by IP prefixes.

- **Game development:** Binary search for collision detection in sorted spatial
  data structures.

- **Operating systems:** Jump search principles in page tables for memory management.

## Common Pitfalls and Best Practices

1. **Integer overflow in midpoint calculation.** Use `mid = low + (high - low) / 2`
   instead of `mid = (low + high) / 2` to avoid overflow when low and high are large.

2. **Off-by-one errors.** Binary search boundary conditions are notoriously tricky.
   Verify that low <= high is the correct loop condition and that low/high updates
   use mid +/- 1, not mid itself.

3. **Applying binary search to unsorted data.** Binary search requires sorted input.
   Sorting first costs O(n log n), so only do this if you search multiple times.

4. **Using interpolation search on skewed data.** The O(log log n) average only holds
   for uniform distributions. Exponentially distributed data can cause O(n) behavior.

5. **Forgetting edge cases.** Handle empty arrays, single-element arrays, and targets
   that are smaller or larger than all elements.

6. **Choosing the wrong variant.** Binary search has many variants: find first
   occurrence, find last occurrence, find insertion point. Use the correct one.

## Practice Problems

1. Modify binary search to return the index of the first occurrence of the target
   in an array that may contain duplicates.

2. Implement binary search to find the insertion point (the index where the target
   should be inserted to maintain sorted order).

3. Given a sorted and rotated array [4, 5, 6, 7, 0, 1, 2], design an O(log n)
   search algorithm that finds a target element.

4. Prove that binary search on a sorted array of n elements requires at most
   CEIL(log2(n + 1)) comparisons.

5. Design an exponential search for an unbounded sorted sequence where you can
   only access elements by index. Analyze the total number of comparisons if the
   target is at position k.

6. Compare the practical performance of jump search and binary search on a sorted
   array stored on disk, where each random access has a fixed latency. Under what
   conditions might jump search with sequential reads outperform binary search?

7. Implement a recursive version of binary search and compare its space complexity
   with the iterative version due to call stack usage.
