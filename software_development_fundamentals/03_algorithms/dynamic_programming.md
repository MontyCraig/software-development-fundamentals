# Dynamic Programming

## Overview

Dynamic programming (DP) is an optimization technique that solves complex problems by
breaking them into simpler overlapping subproblems, solving each subproblem just once,
and storing the results to avoid redundant computation. It transforms exponential-time
recursive solutions into polynomial-time algorithms by exploiting two key properties:
optimal substructure and overlapping subproblems.

There are two main approaches to dynamic programming: top-down (memoization), which
starts from the original problem and recursively solves subproblems while caching
results, and bottom-up (tabulation), which iteratively builds solutions from the
smallest subproblems up to the original problem. Both achieve the same asymptotic
complexity, but differ in implementation style, memory usage, and practical performance.

Dynamic programming is one of the most powerful algorithmic techniques, applicable to
optimization problems in scheduling, resource allocation, sequence alignment, graph
shortest paths, and many other domains. Mastering it requires recognizing the DP
structure in a problem, defining the state and recurrence relation, and choosing the
right implementation strategy.

## Key Concepts

### Optimal Substructure

A problem has optimal substructure if an optimal solution can be constructed from
optimal solutions to its subproblems. For example, the shortest path from A to C
through B consists of the shortest path from A to B plus the shortest path from B to C.

### Overlapping Subproblems

A problem has overlapping subproblems if the same subproblems are solved multiple times
during recursion. Fibonacci is the classic example: computing fib(5) requires fib(3)
twice and fib(2) three times.

### Top-Down vs Bottom-Up

```
Top-Down (Memoization):                Bottom-Up (Tabulation):
- Start with the original problem       - Start with the smallest subproblems
- Recursively break into subproblems    - Iteratively build up solutions
- Cache results in a memo table         - Fill a table from base cases up
- Solves only needed subproblems        - Solves all subproblems in order
- Uses call stack (recursion)           - Uses loops (iteration)
- Risk of stack overflow for deep n     - No stack overflow risk
```

## Algorithms and Pseudocode

### 1. Fibonacci Numbers

**Top-Down (Memoization):**

```
FUNCTION fib_top_down(n: Integer, memo: Map) -> Integer:
    IF n <= 1 THEN
        RETURN n
    END IF
    IF memo CONTAINS n THEN
        RETURN memo[n]
    END IF
    SET memo[n] = fib_top_down(n - 1, memo) + fib_top_down(n - 2, memo)
    RETURN memo[n]
END FUNCTION
```

**Bottom-Up (Tabulation):**

```
FUNCTION fib_bottom_up(n: Integer) -> Integer:
    IF n <= 1 THEN
        RETURN n
    END IF
    SET table = NEW Array[n + 1]
    SET table[0] = 0
    SET table[1] = 1
    FOR i = 2 TO n DO
        SET table[i] = table[i - 1] + table[i - 2]
    END FOR
    RETURN table[n]
END FUNCTION
```

**Space-Optimized Bottom-Up:**

```
FUNCTION fib_optimized(n: Integer) -> Integer:
    IF n <= 1 THEN
        RETURN n
    END IF
    SET prev2 = 0
    SET prev1 = 1
    FOR i = 2 TO n DO
        SET current = prev1 + prev2
        SET prev2 = prev1
        SET prev1 = current
    END FOR
    RETURN prev1
END FUNCTION
```

**Tabulation trace for fib_bottom_up(7):**

```
Index:   0    1    2    3    4    5    6    7
Table: [ 0 ][ 1 ][   ][   ][   ][   ][   ][   ]

i=2: table[2] = table[1] + table[0] = 1 + 0 = 1
     [ 0 ][ 1 ][ 1 ][   ][   ][   ][   ][   ]

i=3: table[3] = table[2] + table[1] = 1 + 1 = 2
     [ 0 ][ 1 ][ 1 ][ 2 ][   ][   ][   ][   ]

i=4: table[4] = table[3] + table[2] = 2 + 1 = 3
     [ 0 ][ 1 ][ 1 ][ 2 ][ 3 ][   ][   ][   ]

i=5: table[5] = table[4] + table[3] = 3 + 2 = 5
     [ 0 ][ 1 ][ 1 ][ 2 ][ 3 ][ 5 ][   ][   ]

i=6: table[6] = table[5] + table[4] = 5 + 3 = 8
     [ 0 ][ 1 ][ 1 ][ 2 ][ 3 ][ 5 ][ 8 ][   ]

i=7: table[7] = table[6] + table[5] = 8 + 5 = 13
     [ 0 ][ 1 ][ 1 ][ 2 ][ 3 ][ 5 ][ 8 ][ 13]

Result: 13
```

### 2. 0/1 Knapsack Problem

Given n items with weights and values, and a knapsack with capacity W, find the
maximum value that can be carried without exceeding the weight limit.

```
FUNCTION knapsack(weights: Array, values: Array, n: Integer, W: Integer) -> Integer:
    // table[i][w] = max value using first i items with capacity w
    SET table = NEW 2D Array[n + 1][W + 1] initialized to 0

    FOR i = 1 TO n DO
        FOR w = 0 TO W DO
            IF weights[i - 1] <= w THEN
                // Choose max of: skip item i, or take item i
                SET take = values[i - 1] + table[i - 1][w - weights[i - 1]]
                SET skip = table[i - 1][w]
                SET table[i][w] = MAX(take, skip)
            ELSE
                SET table[i][w] = table[i - 1][w]      // cannot take item i
            END IF
        END FOR
    END FOR

    RETURN table[n][W]
END FUNCTION
```

**Trace for items: weights=[1,3,4,5], values=[1,4,5,7], W=7:**

```
Table construction (rows = items, columns = capacity):

        w=0  w=1  w=2  w=3  w=4  w=5  w=6  w=7
i=0:  [  0    0    0    0    0    0    0    0  ]  (no items)
i=1:  [  0    1    1    1    1    1    1    1  ]  (item: w=1, v=1)
i=2:  [  0    1    1    4    5    5    5    5  ]  (item: w=3, v=4)
i=3:  [  0    1    1    4    5    6    6    9  ]  (item: w=4, v=5)
i=4:  [  0    1    1    4    5    7    8    9  ]  (item: w=5, v=7)

Answer: table[4][7] = 9

Tracing back: items 2 (v=4,w=3) and 3 (v=5,w=4) give value 9, weight 7.
```

### 3. Longest Common Subsequence (LCS)

Find the longest subsequence common to two sequences.

```
FUNCTION lcs(X: Sequence, Y: Sequence, m: Integer, n: Integer) -> Integer:
    // table[i][j] = length of LCS of X[0..i-1] and Y[0..j-1]
    SET table = NEW 2D Array[m + 1][n + 1] initialized to 0

    FOR i = 1 TO m DO
        FOR j = 1 TO n DO
            IF X[i - 1] = Y[j - 1] THEN
                SET table[i][j] = table[i - 1][j - 1] + 1
            ELSE
                SET table[i][j] = MAX(table[i - 1][j], table[i][j - 1])
            END IF
        END FOR
    END FOR

    RETURN table[m][n]
END FUNCTION
```

**Trace for X = "ABCB", Y = "BDCAB":**

```
        ""   B    D    C    A    B
  "" [  0    0    0    0    0    0  ]
  A  [  0    0    0    0    1    1  ]
  B  [  0    1    1    1    1    2  ]
  C  [  0    1    1    2    2    2  ]
  B  [  0    1    1    2    2    3  ]

LCS length = 3.

Backtrack to find the LCS:
  table[4][5]=3, X[3]=B=Y[4] -> match, go diagonal
  table[3][4]=2, X[2]=C!=Y[3]=A -> go to max(table[2][4], table[3][3])
  table[3][3]=2, X[2]=C=Y[2] -> match, go diagonal
  table[2][2]=1, X[1]=B!=Y[1]=D -> go to max(table[1][2], table[2][1])
  table[2][1]=1, X[1]=B=Y[0] -> match, go diagonal
  table[1][0]=0 -> done

LCS = "BCB" (reading matches in reverse: B, C, B)
```

### 4. Coin Change Problem

Find the minimum number of coins needed to make a given amount.

```
FUNCTION coin_change(coins: Array, n: Integer, amount: Integer) -> Integer:
    // table[a] = minimum coins needed to make amount a
    SET table = NEW Array[amount + 1] initialized to INFINITY
    SET table[0] = 0                        // base case: 0 coins for amount 0

    FOR a = 1 TO amount DO
        FOR i = 0 TO n - 1 DO
            IF coins[i] <= a THEN
                SET table[a] = MIN(table[a], table[a - coins[i]] + 1)
            END IF
        END FOR
    END FOR

    IF table[amount] = INFINITY THEN
        RETURN -1                           // impossible
    END IF
    RETURN table[amount]
END FUNCTION
```

**Trace for coins = [1, 3, 4], amount = 6:**

```
a=0: table[0] = 0   (base case)
a=1: coin 1: table[1-1]+1 = 0+1 = 1.  table[1] = 1
a=2: coin 1: table[2-1]+1 = 1+1 = 2.  table[2] = 2
a=3: coin 1: table[3-1]+1 = 2+1 = 3.
     coin 3: table[3-3]+1 = 0+1 = 1.  table[3] = 1
a=4: coin 1: table[4-1]+1 = 1+1 = 2.
     coin 3: table[4-3]+1 = 1+1 = 2.
     coin 4: table[4-4]+1 = 0+1 = 1.  table[4] = 1
a=5: coin 1: table[5-1]+1 = 1+1 = 2.
     coin 3: table[5-3]+1 = 2+1 = 3.
     coin 4: table[5-4]+1 = 1+1 = 2.  table[5] = 2
a=6: coin 1: table[6-1]+1 = 2+1 = 3.
     coin 3: table[6-3]+1 = 1+1 = 2.
     coin 4: table[6-4]+1 = 2+1 = 3.  table[6] = 2

Table: [0, 1, 2, 1, 1, 2, 2]
Answer: 2 coins (using two coins of value 3: 3 + 3 = 6)
```

## Visual Walkthrough

### Top-Down vs Bottom-Up Computation Order

```
Top-Down for fib(5) -- computes only what is needed:

fib(5) -> fib(4) -> fib(3) -> fib(2) -> fib(1) [base]
                                      -> fib(0) [base]
                           -> fib(1) [base]
                  -> fib(2) [CACHED]
          -> fib(3) [CACHED]

Order of first computation: fib(0), fib(1), fib(2), fib(3), fib(4), fib(5)
Subproblems solved: fib(0), fib(1), fib(2), fib(3), fib(4), fib(5)

Bottom-Up for fib(5) -- computes everything in order:

table: [0] [1] [?] [?] [?] [?]
         i=2: [0] [1] [1] [?] [?] [?]
         i=3: [0] [1] [1] [2] [?] [?]
         i=4: [0] [1] [1] [2] [3] [?]
         i=5: [0] [1] [1] [2] [3] [5]

Order of computation: fib(0), fib(1), fib(2), fib(3), fib(4), fib(5)
Same subproblems, but always computed left-to-right.
```

### Knapsack Decision Diagram

```
Items: (w=2,v=3), (w=3,v=4), (w=4,v=5)   Capacity W=5

Decision tree:
                    [cap=5, val=0]
                   /               \
            Take item 1           Skip item 1
           [cap=3, val=3]         [cap=5, val=0]
           /           \           /           \
     Take item 2   Skip 2    Take item 2   Skip 2
    [cap=0,val=7] [cap=3,v=3] [cap=2,v=4] [cap=5,v=0]
        |           |           |             |
    Skip 3      Take 3?     Skip 3        Take 3?
    val=7      cap<4:NO     val=4        [cap=1,v=5]
                  |                        Skip 3
              Skip 3                      val=5
              val=3

Maximum value: 7 (take items 1 and 2, weight = 2+3 = 5)

DP avoids recomputing overlapping subtrees.
```

### LCS Table Backtracking

```
X = "ABCB", Y = "BDCAB"

        ""   B    D    C    A    B
  "" [  0    0    0    0    0    0  ]
  A  [  0    0    0    0   (1)   1  ]
  B  [  0   (1)   1    1    1    2  ]
  C  [  0    1    1   (2)   2    2  ]
  B  [  0    1    1    2    2   (3) ]

Arrows show backtrack path:
  (3) at [4][5]: B=B match, go to [3][4]
  (2) at [3][3]: C=C match, go to [2][2]
  (1) at [2][1]: B=B match, go to [1][0]
  [1][0] = 0: done

LCS = "BCB" (reverse order of matches)
```

## Complexity Analysis

| Problem           | Naive Recursion | DP Time      | DP Space     | Optimized Space |
|-------------------|-----------------|--------------|--------------|-----------------|
| Fibonacci         | O(2^n)          | O(n)         | O(n)         | O(1)            |
| 0/1 Knapsack      | O(2^n)          | O(n * W)     | O(n * W)     | O(W)            |
| LCS               | O(2^(m+n))      | O(m * n)     | O(m * n)     | O(min(m, n))    |
| Coin Change       | O(amount^n)     | O(n * amount)| O(amount)    | O(amount)       |
| Edit Distance     | O(3^(m+n))      | O(m * n)     | O(m * n)     | O(min(m, n))    |
| Matrix Chain Mult | O(2^n)          | O(n^3)       | O(n^2)       | O(n^2)          |

## When to Use Dynamic Programming

**DP is appropriate when:**
- The problem has optimal substructure
- There are overlapping subproblems
- You need the globally optimal solution (not just locally good)
- The state space is polynomial in size

**DP is NOT appropriate when:**
- Subproblems do not overlap (use divide and conquer instead)
- The problem lacks optimal substructure
- The state space is too large (exponential number of distinct states)
- A greedy approach gives proven optimal results (use greedy instead)

## Real-World Applications

- **Bioinformatics:** Sequence alignment algorithms (Smith-Waterman, Needleman-Wunsch)
  use DP to find optimal alignments between DNA or protein sequences.

- **Natural language processing:** The Viterbi algorithm uses DP for hidden Markov
  model decoding in speech recognition and part-of-speech tagging.

- **Finance:** Portfolio optimization, option pricing, and resource allocation use
  DP to maximize returns under constraints.

- **Networking:** Shortest path algorithms (Bellman-Ford, Floyd-Warshall) are DP
  algorithms for routing packets through networks.

- **Text processing:** Edit distance (Levenshtein distance) powers spell checkers,
  DNA comparison, and diff utilities.

## Common Pitfalls and Best Practices

1. **Incorrect state definition.** The state must capture all information needed to
   make optimal decisions. Missing a dimension leads to wrong answers.

2. **Wrong base cases.** DP tables must be initialized correctly. Off-by-one errors
   in base cases propagate through the entire table.

3. **Not recognizing DP applicability.** If you see overlapping subproblems in a
   recursion tree, that is a strong signal to apply DP.

4. **Excessive memory usage.** Many DP problems can use space-optimized versions
   that keep only the previous row or a few variables instead of the full table.

5. **Forgetting to handle impossible states.** Use INFINITY or special sentinel
   values for states that cannot be reached, and check for them in the final answer.

6. **Top-down vs bottom-up choice.** Top-down is easier to write but uses stack
   space. Bottom-up is more efficient but requires careful ordering of subproblems.

## Practice Problems

1. Implement the edit distance algorithm to compute the minimum number of insertions,
   deletions, and substitutions to transform one string into another.

2. Solve the matrix chain multiplication problem: given dimensions of n matrices,
   find the order of multiplication that minimizes total scalar multiplications.

3. Implement the longest increasing subsequence (LIS) problem using DP in O(n^2)
   time. Then describe how to improve it to O(n log n) using binary search.

4. Solve the rod cutting problem: given a rod of length n and a price table, find
   the maximum revenue from cutting the rod into pieces.

5. Implement the partition problem: determine if a set of integers can be divided
   into two subsets with equal sum.

6. Solve the egg drop problem: given k eggs and n floors, find the minimum number
   of trials needed to determine the critical floor.

7. Convert the top-down knapsack solution to space-optimized bottom-up using a
   single 1D array. Explain why the inner loop must run backwards.
