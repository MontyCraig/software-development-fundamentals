# Recursion and Backtracking

## Overview

Recursion is a problem-solving strategy in which a function solves a problem by
calling itself on smaller instances of the same problem. Every recursive solution
requires a base case that terminates the recursion and a recursive case that breaks
the problem down. While recursion often leads to elegant and concise solutions, it
carries overhead from function call management and can lead to stack overflow if the
recursion depth is too deep.

Backtracking is a systematic method for exploring all possible solutions by building
candidates incrementally and abandoning ("pruning") a candidate as soon as it is
determined that it cannot lead to a valid solution. Backtracking uses recursion as its
primary mechanism, but adds the crucial step of undoing choices (backtracking) when a
dead end is reached.

Together, recursion and backtracking are essential tools for solving constraint
satisfaction problems, combinatorial puzzles, and optimization problems where brute
force would be prohibitively expensive. This chapter covers the mechanics of both
techniques, including call stack visualization, memoization, and classic problems.

## Key Concepts

### The Anatomy of a Recursive Function

Every recursive function has exactly two parts:

```
FUNCTION recursive_example(input: Value) -> Result:
    // BASE CASE: Directly return a result for the smallest subproblem
    IF input is trivial THEN
        RETURN trivial_result
    END IF

    // RECURSIVE CASE: Reduce the problem and call self
    SET smaller = reduce(input)
    SET sub_result = recursive_example(smaller)
    RETURN combine(sub_result)
END FUNCTION
```

### The Call Stack

Each recursive call creates a new frame on the call stack containing local variables,
parameters, and the return address. The stack unwinds as each call returns.

### Memoization

Memoization stores the results of expensive function calls and returns the cached
result when the same inputs occur again, converting exponential-time recursion into
polynomial-time dynamic programming.

## Algorithms and Pseudocode

### 1. Factorial (Basic Recursion)

```
FUNCTION factorial(n: Integer) -> Integer:
    IF n <= 1 THEN
        RETURN 1                            // base case
    END IF
    RETURN n * factorial(n - 1)             // recursive case
END FUNCTION
```

**Call stack trace for factorial(5):**

```
Call Stack Growth:                    Unwinding:

factorial(5)                          factorial(5) = 5 * 24 = 120
  factorial(4)                          factorial(4) = 4 * 6 = 24
    factorial(3)                          factorial(3) = 3 * 2 = 6
      factorial(2)                          factorial(2) = 2 * 1 = 2
        factorial(1)                          factorial(1) = 1 (base)

+-------------------+
| factorial(1)      |  <-- top of stack
| n=1, return 1     |
+-------------------+
| factorial(2)      |
| n=2, waiting...   |
+-------------------+
| factorial(3)      |
| n=3, waiting...   |
+-------------------+
| factorial(4)      |
| n=4, waiting...   |
+-------------------+
| factorial(5)      |  <-- bottom of stack
| n=5, waiting...   |
+-------------------+
```

### 2. Fibonacci (Demonstrating Memoization Need)

**Naive recursive version (exponential time):**

```
FUNCTION fibonacci(n: Integer) -> Integer:
    IF n <= 1 THEN
        RETURN n
    END IF
    RETURN fibonacci(n - 1) + fibonacci(n - 2)
END FUNCTION
```

**Call tree for fibonacci(5) showing redundant computation:**

```
                          fib(5)
                        /        \
                   fib(4)         fib(3)
                  /     \         /    \
              fib(3)   fib(2)  fib(2)  fib(1)
             /    \    /   \    /   \      |
          fib(2) fib(1) fib(1) fib(0) fib(1) fib(0) 1
          /   \    |     |      |      |      |
       fib(1) fib(0) 1   1      0      1      0
          |      |
          1      0

fib(2) is computed 3 times! fib(3) is computed 2 times!
Total calls: 15 for n=5. For n=50 this would be billions.
```

**Memoized version (linear time):**

```
FUNCTION fibonacci_memo(n: Integer, memo: Map) -> Integer:
    IF n <= 1 THEN
        RETURN n
    END IF
    IF memo CONTAINS n THEN
        RETURN memo[n]
    END IF
    SET result = fibonacci_memo(n - 1, memo) + fibonacci_memo(n - 2, memo)
    SET memo[n] = result
    RETURN result
END FUNCTION
```

**Memoized call trace for fibonacci_memo(5):**

```
fib(5) -> needs fib(4) and fib(3)
  fib(4) -> needs fib(3) and fib(2)
    fib(3) -> needs fib(2) and fib(1)
      fib(2) -> needs fib(1) and fib(0)
        fib(1) = 1 (base)
        fib(0) = 0 (base)
      fib(2) = 1  [stored in memo]
      fib(1) = 1 (base)
    fib(3) = 2  [stored in memo]
    fib(2) = 1  [retrieved from memo -- no recomputation]
  fib(4) = 3  [stored in memo]
  fib(3) = 2  [retrieved from memo -- no recomputation]
fib(5) = 5

Only 9 calls instead of 15. For large n the savings are exponential.
```

### 3. Recursive vs Iterative Comparison

```
// Recursive sum
FUNCTION sum_recursive(A: Array, n: Integer) -> Integer:
    IF n = 0 THEN
        RETURN 0
    END IF
    RETURN A[n - 1] + sum_recursive(A, n - 1)
END FUNCTION

// Iterative sum
FUNCTION sum_iterative(A: Array, n: Integer) -> Integer:
    SET total = 0
    FOR i = 0 TO n - 1 DO
        SET total = total + A[i]
    END FOR
    RETURN total
END FUNCTION
```

**Comparison:**

| Aspect           | Recursive                | Iterative                |
|-----------------|--------------------------|--------------------------|
| Time             | O(n)                     | O(n)                     |
| Space            | O(n) call stack          | O(1)                     |
| Readability      | Often more intuitive     | More explicit             |
| Stack overflow   | Risk for large n         | No risk                  |
| Tail optimization| Possible if supported    | Not applicable            |

### 4. The Backtracking Template

```
FUNCTION backtrack(state: State, choices: List):
    IF state is a complete solution THEN
        RECORD or RETURN state
        RETURN
    END IF

    FOR EACH choice IN available_choices(state) DO
        IF is_valid(state, choice) THEN        // pruning condition
            MAKE choice (modify state)
            backtrack(state, remaining_choices)
            UNDO choice (restore state)         // backtrack
        END IF
    END FOR
END FUNCTION
```

**Key elements:**
- **State:** Current partial solution
- **Choices:** Options available at this step
- **Constraint check (pruning):** Skip invalid branches early
- **Make/Undo:** Modify state before recursing, restore after

### 5. N-Queens Problem

Place n queens on an n x n board so that no two queens attack each other (same row,
column, or diagonal).

```
FUNCTION solve_n_queens(n: Integer) -> List of Solutions:
    SET solutions = EMPTY LIST
    SET board = NEW Array[n] initialized to -1   // board[row] = column of queen
    place_queen(board, 0, n, solutions)
    RETURN solutions
END FUNCTION

FUNCTION place_queen(board: Array, row: Integer, n: Integer, solutions: List):
    IF row = n THEN
        ADD COPY(board) TO solutions            // all queens placed
        RETURN
    END IF

    FOR col = 0 TO n - 1 DO
        IF is_safe(board, row, col) THEN
            SET board[row] = col                // place queen
            place_queen(board, row + 1, n, solutions)
            SET board[row] = -1                 // remove queen (backtrack)
        END IF
    END FOR
END FUNCTION

FUNCTION is_safe(board: Array, row: Integer, col: Integer) -> Boolean:
    FOR i = 0 TO row - 1 DO
        IF board[i] = col THEN
            RETURN false                        // same column
        END IF
        IF ABS(board[i] - col) = ABS(i - row) THEN
            RETURN false                        // same diagonal
        END IF
    END FOR
    RETURN true
END FUNCTION
```

**Step-by-step trace for 4-Queens:**

```
Board (4x4), placing queens row by row:

Row 0: Try col 0
  Row 1: Try col 0 (same col) SKIP
         Try col 1 (diagonal) SKIP
         Try col 2 -> PLACE
    Row 2: Try col 0 (diagonal from row1) SKIP
           Try col 1 (diagonal from row0) SKIP
           Try col 2 (same col) SKIP
           Try col 3 (diagonal from row1) SKIP
           DEAD END -> BACKTRACK row 1
         Try col 3 -> PLACE
    Row 2: Try col 0 (diagonal) SKIP
           Try col 1 -> PLACE
      Row 3: Try col 0 (diagonal) SKIP
             Try col 1 (same col) SKIP
             Try col 2 (diagonal) SKIP
             Try col 3 (same col, diagonal) SKIP
             DEAD END -> BACKTRACK row 2
           Try col 2 (diagonal) SKIP
           Try col 3 (same col) SKIP
           DEAD END -> BACKTRACK row 1
         All cols tried -> BACKTRACK row 0

Row 0: Try col 1
  Row 1: Try col 0 (diagonal) SKIP
         Try col 1 (same col) SKIP
         Try col 2 (diagonal) SKIP
         Try col 3 -> PLACE
    Row 2: Try col 0 -> PLACE
      Row 3: Try col 0 (same col) SKIP
             Try col 1 (same col, diagonal) SKIP
             Try col 2 -> PLACE -> SOLUTION FOUND!

Solution 1:
  . Q . .      board = [1, 3, 0, 2]
  . . . Q
  Q . . .
  . . Q .
```

### Backtracking Search Tree Visualization (4-Queens)

```
                              Start
                    /      |      |      \
               Q@(0,0)  Q@(0,1)  Q@(0,2)  Q@(0,3)
              /  |  \     / | \
         (1,2) (1,3)  (1,3)  ...
          / \     |       |
       FAIL FAIL (2,1)  (2,0)
                  |       |
                FAIL    (3,2) -> SOLUTION [1,3,0,2]

Pruned branches shown as FAIL.
Without pruning: 4^4 = 256 nodes to explore.
With pruning: only ~24 nodes explored (90%+ reduction).
```

## Visual Walkthrough

### Recursion as a Stack of Frames

```
FUNCTION power(base: Integer, exp: Integer) -> Integer:
    IF exp = 0 THEN RETURN 1 END IF
    RETURN base * power(base, exp - 1)
END FUNCTION

Call: power(2, 4)

Stack during deepest call:        Stack during unwinding:

+------------------+
| power(2, 0)      | -> returns 1
+------------------+
| power(2, 1)      | -> 2 * 1 = 2
+------------------+
| power(2, 2)      | -> 2 * 2 = 4
+------------------+
| power(2, 3)      | -> 2 * 4 = 8
+------------------+
| power(2, 4)      | -> 2 * 8 = 16
+------------------+

Depth = exp = 4, Space = O(exp)
```

### Memoization: Cache Hit Visualization

```
Computing fibonacci_memo(6):

Call sequence:
fib(6) -> fib(5) -> fib(4) -> fib(3) -> fib(2) -> fib(1) [base]
                                                -> fib(0) [base]
                                         memo[2] = 1
                                -> fib(1) [base]
                         memo[3] = 2
                  -> fib(2) [CACHE HIT]
           memo[4] = 3
    -> fib(3) [CACHE HIT]
memo[5] = 5
-> fib(4) [CACHE HIT]
memo[6] = 8

Memo table state:
+-----+---+---+---+---+---+---+---+
| key | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
+-----+---+---+---+---+---+---+---+
| val | 0 | 1 | 1 | 2 | 3 | 5 | 8 |
+-----+---+---+---+---+---+---+---+

Without memo: 25 calls.  With memo: 11 calls.
```

## Complexity Analysis

| Problem                | Without Memo    | With Memo   | Space    |
|-----------------------|-----------------|-------------|----------|
| Fibonacci             | O(2^n)          | O(n)        | O(n)     |
| Factorial             | O(n)            | O(n)        | O(n)     |
| N-Queens              | O(n!)           | N/A         | O(n)     |
| Subset Sum            | O(2^n)          | O(n * sum)  | O(n*sum) |
| Permutations          | O(n * n!)       | N/A         | O(n)     |
| Tower of Hanoi        | O(2^n)          | N/A         | O(n)     |

### Backtracking Pruning Effectiveness

```
N-Queens brute force vs backtracking:

n    | Brute Force (n^n) | Backtracking | Reduction
-----|-------------------|--------------|----------
4    | 256               | ~24          | 90.6%
8    | 16,777,216        | ~15,720      | 99.9%
12   | 8.9 x 10^12      | ~856,188     | ~100%

Pruning transforms intractable problems into solvable ones.
```

## When to Use Each Technique

- **Simple Recursion:** Natural fit for problems defined in terms of smaller
  subproblems -- tree traversals, mathematical recurrences, divide and conquer.

- **Memoization:** When recursive solutions have overlapping subproblems (the same
  inputs are computed multiple times). Converts exponential to polynomial time.

- **Backtracking:** Constraint satisfaction problems where you must search through
  a space of possible configurations -- puzzles, scheduling, combinatorial optimization.

- **Iterative Approach:** When recursion depth could cause stack overflow, when
  tail recursion is not optimized, or when the iterative version is equally clear.

## Real-World Applications

- **Compiler parsing:** Recursive descent parsers process grammar rules by recursive
  function calls matching production rules.

- **File system traversal:** Recursively listing directories mirrors the tree structure
  of file systems naturally.

- **Puzzle solvers:** Backtracking solves Sudoku, crossword puzzles, and constraint
  satisfaction problems used in AI planning.

- **Game AI:** Minimax with alpha-beta pruning (a form of backtracking) powers
  decision-making in two-player games.

- **Regular expression matching:** Backtracking engines try different matching
  possibilities and backtrack on failures.

- **Configuration testing:** Backtracking explores valid combinations of system
  configurations while pruning incompatible settings.

## Common Pitfalls and Best Practices

1. **Missing base case.** Without a proper base case, recursion runs forever until
   the call stack overflows. Always define and test base cases first.

2. **Redundant computation.** Naive recursion on problems with overlapping
   subproblems (like Fibonacci) causes exponential blowup. Use memoization.

3. **Excessive stack depth.** Deep recursion (depth > 10,000) risks stack overflow.
   Consider iterative alternatives or tail-call optimization where available.

4. **Not restoring state in backtracking.** After exploring a branch, you must undo
   the choice completely. Failing to restore state corrupts subsequent branches.

5. **Weak pruning conditions.** Effective backtracking depends on pruning bad branches
   early. Analyze constraints carefully to maximize pruning.

6. **Using recursion where iteration suffices.** Simple loops should remain loops.
   Recursion adds overhead and is warranted only when the problem structure benefits.

## Practice Problems

1. Write a recursive function to compute the sum of digits of a positive integer.
   Trace the call stack for input 9742.

2. Implement a recursive function to generate all permutations of an array
   [1, 2, 3]. Show the backtracking tree.

3. Solve the subset sum problem: given a set of integers, find all subsets that
   sum to a target value. Use backtracking with pruning.

4. Convert the recursive Fibonacci function to an iterative version using a loop.
   Compare the space complexity of both approaches.

5. Implement a Sudoku solver using backtracking. Define the pruning conditions
   for row, column, and 3x3 box constraints.

6. Write a recursive function to check whether a string is a palindrome. Then
   rewrite it iteratively and compare both approaches.

7. Design a backtracking algorithm to find all ways to place n rooks on an n x n
   chessboard such that no two rooks attack each other. How does this differ from
   the N-Queens problem?

8. Implement the Tower of Hanoi for n disks. Prove that the minimum number of
   moves required is 2^n - 1 using mathematical induction.
