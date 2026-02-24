# Complexity Analysis

## Overview

Complexity analysis is the mathematical framework used to evaluate the efficiency of
algorithms in terms of time and space as the input size grows. Rather than measuring
wall-clock execution time, which varies across hardware and environments, complexity
analysis provides a machine-independent way to compare algorithms by describing how
their resource consumption scales with input size.

The three principal notations -- Big O, Big Omega, and Big Theta -- capture upper bounds,
lower bounds, and tight bounds respectively. Together with amortized analysis, recurrence
relations, and the master theorem, these tools allow developers to reason precisely about
algorithm performance before writing a single line of code.

Understanding complexity analysis is essential for making informed design decisions. An
algorithm that appears fast on small inputs may become unusable at scale, while one that
seems slower for trivial cases may dominate for large datasets. This chapter equips you
with the analytical tools to tell the difference.

## Key Concepts

### Asymptotic Notation

Asymptotic notation describes algorithm behavior as input size approaches infinity,
stripping away constants and lower-order terms to focus on the dominant growth factor.

**Big O (Upper Bound):** f(n) is O(g(n)) if there exist constants c > 0 and n0 >= 0
such that f(n) <= c * g(n) for all n >= n0. This gives the worst-case ceiling.

**Big Omega (Lower Bound):** f(n) is Omega(g(n)) if there exist constants c > 0 and
n0 >= 0 such that f(n) >= c * g(n) for all n >= n0. This gives the best-case floor.

**Big Theta (Tight Bound):** f(n) is Theta(g(n)) if f(n) is both O(g(n)) and
Omega(g(n)). This means the algorithm grows at exactly the rate of g(n) up to
constant factors.

### Growth Rate Comparison

```
Operations
    |
    |                                                      * 2^n
    |                                                *
    |                                           *
    |                                      *
    |                                  *          . n^2
    |                             *        .
    |                         *       .
    |                     *      .            ~ n log n
    |                  *    .           ~
    |              *   .          ~
    |           *  .        ~              --- n
    |        * .      ~               ---
    |      *.    ~              ---
    |    *. ~            ---               ... log n
    |  *.~        ---            ...
    | *~   ---         ...                 ___ 1
    |*---...___________________________________
    +-------------------------------------------> n
    0    10    20    30    40    50    60    70
```

### Common Complexity Classes

| Class        | Name           | Example Operation                      |
|-------------|----------------|----------------------------------------|
| O(1)        | Constant       | Array index access, hash table lookup  |
| O(log n)    | Logarithmic    | Binary search                          |
| O(n)        | Linear         | Single pass through array              |
| O(n log n)  | Linearithmic   | Efficient comparison sort              |
| O(n^2)      | Quadratic      | Nested iteration over pairs            |
| O(n^3)      | Cubic          | Triple nested loops, matrix multiply   |
| O(2^n)      | Exponential    | All subsets, naive recursive Fibonacci  |
| O(n!)       | Factorial      | All permutations, brute-force TSP      |

## Algorithms and Pseudocode

### Counting Primitive Operations

To determine complexity, count how many primitive operations execute as a function
of input size n.

```
FUNCTION sum_of_array(A: Array of Integer, n: Integer) -> Integer:
    SET total = 0                   // 1 operation
    FOR i = 0 TO n - 1 DO          // loop runs n times
        SET total = total + A[i]   // 2 operations per iteration (add + access)
    END FOR
    RETURN total                    // 1 operation
END FUNCTION

// Total: 1 + n * 2 + 1 = 2n + 2 = O(n)
```

### Nested Loop Analysis

```
FUNCTION find_pair_sum(A: Array of Integer, n: Integer, target: Integer) -> Boolean:
    FOR i = 0 TO n - 2 DO                      // outer: n - 1 iterations
        FOR j = i + 1 TO n - 1 DO              // inner: n - i - 1 iterations
            IF A[i] + A[j] = target THEN       // 1 comparison
                RETURN true
            END IF
        END FOR
    END FOR
    RETURN false
END FUNCTION

// Total comparisons: (n-1) + (n-2) + ... + 1 = n(n-1)/2 = O(n^2)
```

### Logarithmic Example

```
FUNCTION count_halvings(n: Integer) -> Integer:
    SET count = 0
    SET value = n
    WHILE value > 1 DO
        SET value = FLOOR(value / 2)
        SET count = count + 1
    END WHILE
    RETURN count
END FUNCTION

// Each iteration halves value, so iterations = floor(log2(n)) = O(log n)
```

## Amortized Analysis

Amortized analysis computes the average cost per operation over a sequence of
operations, even when individual operations vary widely in cost.

### Dynamic Array Append Example

```
FUNCTION append(A: DynamicArray, element: Value):
    IF A.size = A.capacity THEN
        SET new_capacity = A.capacity * 2
        SET B = NEW Array[new_capacity]
        FOR i = 0 TO A.size - 1 DO
            SET B[i] = A[i]                     // copy all n elements: O(n)
        END FOR
        SET A.storage = B
        SET A.capacity = new_capacity
    END IF
    SET A.storage[A.size] = element             // O(1)
    SET A.size = A.size + 1
END FUNCTION
```

**Step-by-step cost trace for appending elements 1 through 9:**

```
Append #  | Capacity Before | Copy Cost | Insert Cost | Total
----------|-----------------|-----------|-------------|------
    1     |       1         |     0     |      1      |   1
    2     |       1 -> 2    |     1     |      1      |   2
    3     |       2 -> 4    |     2     |      1      |   3
    4     |       4         |     0     |      1      |   1
    5     |       4 -> 8    |     4     |      1      |   5
    6     |       8         |     0     |      1      |   1
    7     |       8         |     0     |      1      |   1
    8     |       8         |     0     |      1      |   1
    9     |       8 -> 16   |     8     |      1      |   9
----------|-----------------|-----------|-------------|------
                          Total cost for 9 appends:     24
                          Amortized cost per append:    24/9 ~ 2.67 = O(1)
```

Although individual resizes cost O(n), they happen so infrequently that the
amortized cost per append is O(1).

## Recurrence Relations

Many recursive algorithms have running times described by recurrence relations.

### Common Recurrences

**Linear recursion (single subproblem, linear work):**
```
T(n) = T(n - 1) + O(1)
Solution: T(n) = O(n)
```

**Binary recursion (two subproblems, constant work):**
```
T(n) = 2 * T(n - 1) + O(1)
Solution: T(n) = O(2^n)
```

**Divide and conquer (halving with linear merge):**
```
T(n) = 2 * T(n/2) + O(n)
Solution: T(n) = O(n log n)     // merge sort
```

**Divide and conquer (halving with constant work):**
```
T(n) = T(n/2) + O(1)
Solution: T(n) = O(log n)       // binary search
```

### Solving by Expansion (Merge Sort)

```
T(n) = 2T(n/2) + cn
     = 2[2T(n/4) + cn/2] + cn    = 4T(n/4) + 2cn
     = 4[2T(n/8) + cn/4] + 2cn   = 8T(n/8) + 3cn
     ...
     = 2^k * T(n/2^k) + k * cn

When n/2^k = 1, k = log2(n):
T(n) = n * T(1) + cn * log2(n) = O(n log n)
```

## The Master Theorem

For recurrences of the form T(n) = a * T(n/b) + O(n^d) where a >= 1, b > 1:

```
+-----------+----------------------------+---------------------------+
|  Case     |  Condition                 |  Solution                 |
+-----------+----------------------------+---------------------------+
|  Case 1   |  d < log_b(a)              |  T(n) = O(n^(log_b(a)))   |
|  Case 2   |  d = log_b(a)              |  T(n) = O(n^d * log n)    |
|  Case 3   |  d > log_b(a)              |  T(n) = O(n^d)            |
+-----------+----------------------------+---------------------------+
```

**Example applications:**

| Recurrence             | a  | b  | d  | log_b(a) | Case | Result        |
|------------------------|----|----|----|----------|------|---------------|
| T(n) = 2T(n/2) + n    | 2  | 2  | 1  | 1        | 2    | O(n log n)    |
| T(n) = T(n/2) + 1     | 1  | 2  | 0  | 0        | 2    | O(log n)      |
| T(n) = 4T(n/2) + n    | 4  | 2  | 1  | 2        | 1    | O(n^2)        |
| T(n) = 3T(n/4) + n^2  | 3  | 4  | 2  | ~0.79    | 3    | O(n^2)        |
| T(n) = 8T(n/2) + n^2  | 8  | 2  | 2  | 3        | 1    | O(n^3)        |

## Visual Walkthrough

### Complexity Classes Side by Side (n = 16)

```
n = 16

O(1):       |#                                              |  1 operation
O(log n):   |####                                           |  4 operations
O(n):       |################                               |  16 operations
O(n log n): |################################################################|  64 operations
O(n^2):     would need 256 characters (16x wider than n)
O(2^n):     would need 65,536 characters (impossibly wide)
```

### Recursion Tree for T(n) = 2T(n/2) + n

```
Level 0:              [  n  ]                  cost: n
                     /       \
Level 1:        [n/2]         [n/2]            cost: n
                /   \         /   \
Level 2:    [n/4] [n/4]   [n/4] [n/4]         cost: n
              |     |       |     |
             ...   ...     ...   ...
              |     |       |     |
Level k:    [1] [1] ... [1] [1] [1] [1]       cost: n

Height = log2(n) levels, each level costs n
Total = n * log2(n) = O(n log n)
```

## Space Complexity

Space complexity measures additional memory an algorithm requires beyond its input.

```
FUNCTION reverse_array_in_place(A: Array, n: Integer):
    // Space: O(1) -- only uses a constant number of variables
    FOR i = 0 TO FLOOR(n / 2) - 1 DO
        SET temp = A[i]
        SET A[i] = A[n - 1 - i]
        SET A[n - 1 - i] = temp
    END FOR
END FUNCTION

FUNCTION reverse_array_copy(A: Array, n: Integer) -> Array:
    // Space: O(n) -- creates a new array of size n
    SET B = NEW Array[n]
    FOR i = 0 TO n - 1 DO
        SET B[i] = A[n - 1 - i]
    END FOR
    RETURN B
END FUNCTION
```

## When to Use Each Analysis Technique

| Technique           | Use When                                            |
|--------------------|-----------------------------------------------------|
| Big O              | You need a worst-case upper bound guarantee          |
| Big Omega          | You need to prove a lower bound on any algorithm     |
| Big Theta          | You want an exact characterization of growth         |
| Amortized Analysis | Operations have varying costs over a sequence        |
| Recurrence Solving | Algorithm is recursive with identifiable subproblems |
| Master Theorem     | Recurrence fits the form T(n) = aT(n/b) + O(n^d)    |

## Real-World Applications

- **Database query optimization:** Choose index structures based on O(log n) lookup
  versus O(n) full scan tradeoffs.
- **API rate limiting:** Amortized analysis justifies token bucket algorithms.
- **Memory allocation:** Dynamic arrays use amortized O(1) append.
- **Network routing:** Shortest-path algorithms selected by complexity guarantees.
- **Compiler optimization:** Loop analysis uses complexity to choose transformations.

## Common Pitfalls and Best Practices

1. **Ignoring constants matters at small n.** O(n) with a constant of 1000 is slower
   than O(n^2) for n < 1000. Asymptotic analysis is for large n.

2. **Confusing best case with average case.** An O(n^2) worst-case algorithm might
   have O(n log n) average case -- know which you are measuring.

3. **Forgetting space complexity.** An algorithm with great time complexity but O(n^2)
   space may be impractical for large inputs.

4. **Dropping the wrong terms.** O(n^2 + n) simplifies to O(n^2), but O(n + m)
   cannot simplify further when n and m are independent.

5. **Misapplying the master theorem.** It only applies to recurrences of the
   specific form T(n) = aT(n/b) + O(n^d). Verify the form before applying.

## Practice Problems

1. Determine the time complexity of a function that iterates through an n x n matrix
   and for each cell performs a binary search on a sorted array of length m.

2. Solve the recurrence T(n) = 3T(n/3) + n using the master theorem.

3. An algorithm processes n elements. For each element, it performs an operation
   that costs O(1) normally but O(n) every n-th time. What is the amortized cost?

4. Rank these functions by growth rate: n^2, 2^n, n log n, log(log n), n!, n^(1/2).

5. A function calls itself three times with input size n-1 and does O(1) work at
   each call. Write the recurrence and determine its complexity class.

6. Prove that n^2 + 5n + 12 is Theta(n^2) by finding appropriate constants.

7. An algorithm uses a stack that grows to at most depth log(n), with each stack
   frame holding O(1) data. What is its space complexity?

8. Compare two algorithms: Algorithm A runs in O(n log n) time and O(n) space;
   Algorithm B runs in O(n^2) time and O(1) space. Under what conditions would
   you prefer each?
