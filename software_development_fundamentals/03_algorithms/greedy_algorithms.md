# Greedy Algorithms

## Overview

Greedy algorithms solve optimization problems by making the locally optimal choice at
each step, hoping that these local optima lead to a global optimum. Unlike dynamic
programming, which considers all possible subproblem combinations, a greedy algorithm
commits to each choice immediately and never reconsiders. This makes greedy algorithms
simple to design and efficient to execute, but they only produce correct results when
the problem has the right structural properties.

The two properties that make greedy algorithms work are the greedy choice property
(a globally optimal solution can be reached by making locally optimal choices) and
optimal substructure (an optimal solution contains optimal solutions to subproblems).
When both properties hold, a greedy approach is typically faster and simpler than
dynamic programming. When they do not hold, greedy algorithms may produce suboptimal
or even worst-case results.

This chapter covers foundational greedy algorithms including activity selection,
Huffman coding, and fractional knapsack. It also provides a systematic comparison
with dynamic programming to help you recognize when each approach is appropriate.

## Key Concepts

### The Greedy Choice Property

A problem has the greedy choice property if making the locally best choice at each
step leads to a globally optimal solution. This must be proven for each problem --
it cannot be assumed.

```
Greedy Strategy:
  At each decision point, choose the option that looks best RIGHT NOW.
  Never go back and reconsider previous choices.

  Decision 1 ---> Decision 2 ---> Decision 3 ---> ... ---> Solution
  (best now)     (best now)      (best now)                (optimal?)
```

### Optimal Substructure

After making a greedy choice, the remaining subproblem must also have an optimal
solution that can be found using the same greedy strategy.

### Greedy vs Dynamic Programming

```
+-------------------+---------------------------+---------------------------+
| Property          | Greedy                    | Dynamic Programming       |
+-------------------+---------------------------+---------------------------+
| Choices           | Make one best choice      | Consider all choices      |
| Subproblems       | Solve one subproblem      | Solve many subproblems    |
| Going back        | Never reconsider          | Combine all results       |
| Correctness       | Requires proof            | Always finds optimum      |
| Efficiency        | Often O(n log n) or O(n)  | Often O(n^2) or O(n*W)   |
| Implementation    | Simpler                   | More complex              |
+-------------------+---------------------------+---------------------------+
```

### When Greedy Fails

```
Coin change: coins = [1, 3, 4], amount = 6

Greedy (pick largest first):  4 + 1 + 1 = 6  (3 coins)
Optimal (DP):                 3 + 3 = 6       (2 coins)

The greedy choice (pick 4) leads to a suboptimal solution.
This happens because the coin change problem lacks the greedy choice property
for arbitrary coin denominations.
```

## Algorithms and Pseudocode

### 1. Activity Selection Problem

Given a set of activities with start and finish times, select the maximum number of
non-overlapping activities.

```
FUNCTION activity_selection(activities: Array of (start, finish)) -> List:
    // Sort activities by finish time
    SORT activities BY finish time in ascending order

    SET selected = EMPTY LIST
    ADD activities[0] TO selected
    SET last_finish = activities[0].finish

    FOR i = 1 TO LENGTH(activities) - 1 DO
        IF activities[i].start >= last_finish THEN
            ADD activities[i] TO selected
            SET last_finish = activities[i].finish
        END IF
    END FOR

    RETURN selected
END FUNCTION
```

**Step-by-step trace:**

```
Activities (already sorted by finish time):
  A: [1, 4)    B: [3, 5)    C: [0, 6)    D: [5, 7)
  E: [3, 9)    F: [5, 9)    G: [6, 10)   H: [8, 11)
  I: [8, 12)   J: [2, 14)   K: [12, 16)

Timeline:
0    2    4    6    8    10   12   14   16
|----A----|
     |----B----|
|--------C--------|
               |----D----|
     |--------E----------|
               |----F----|
               |------G--------|
                    |------H--------|
                    |--------I----------|
     |----------J--------------------|
                              |----K----|

Greedy selection:
  Select A [1,4): last_finish = 4
  B [3,5): 3 < 4, skip (overlaps A)
  C [0,6): 0 < 4, skip
  D [5,7): 5 >= 4, SELECT. last_finish = 7
  E [3,9): 3 < 7, skip
  F [5,9): 5 < 7, skip
  G [6,10): 6 < 7, skip
  H [8,11): 8 >= 7, SELECT. last_finish = 11
  I [8,12): 8 < 11, skip
  J [2,14): 2 < 11, skip
  K [12,16): 12 >= 11, SELECT. last_finish = 16

Result: {A, D, H, K} -- 4 activities (optimal)
```

### 2. Huffman Coding

Build an optimal prefix-free binary code for data compression by repeatedly merging
the two lowest-frequency symbols.

```
FUNCTION huffman_coding(symbols: Array of (char, frequency)) -> HuffmanTree:
    // Create a leaf node for each symbol
    SET priority_queue = NEW MinHeap
    FOR EACH (char, freq) IN symbols DO
        INSERT NEW LeafNode(char, freq) INTO priority_queue
    END FOR

    // Build tree by merging two lowest-frequency nodes
    WHILE SIZE(priority_queue) > 1 DO
        SET left = EXTRACT_MIN(priority_queue)
        SET right = EXTRACT_MIN(priority_queue)
        SET parent = NEW InternalNode(
            frequency: left.frequency + right.frequency,
            left: left,
            right: right
        )
        INSERT parent INTO priority_queue
    END WHILE

    RETURN EXTRACT_MIN(priority_queue)      // root of Huffman tree
END FUNCTION

FUNCTION generate_codes(node: HuffmanNode, prefix: String, codes: Map):
    IF node IS LeafNode THEN
        SET codes[node.char] = prefix
        RETURN
    END IF
    generate_codes(node.left, prefix + "0", codes)
    generate_codes(node.right, prefix + "1", codes)
END FUNCTION
```

**Step-by-step trace for symbols: a:5, b:9, c:12, d:13, e:16, f:45:**

```
Initial priority queue: [a:5, b:9, c:12, d:13, e:16, f:45]

Step 1: Merge a(5) + b(9) = ab(14)
  Queue: [c:12, d:13, ab:14, e:16, f:45]

Step 2: Merge c(12) + d(13) = cd(25)
  Queue: [ab:14, e:16, cd:25, f:45]

Step 3: Merge ab(14) + e(16) = abe(30)
  Queue: [cd:25, abe:30, f:45]

Step 4: Merge cd(25) + abe(30) = cdabe(55)
  Queue: [f:45, cdabe:55]

Step 5: Merge f(45) + cdabe(55) = root(100)
  Queue: [root:100]

Huffman Tree:
                  (100)
                 /     \
              f:45    (55)
                     /    \
                  (25)    (30)
                  / \     / \
               c:12 d:13 (14) e:16
                         / \
                       a:5 b:9

Codes:
  f = 0          (1 bit, highest frequency)
  c = 100        (3 bits)
  d = 101        (3 bits)
  a = 1100       (4 bits, lowest frequency)
  b = 1101       (4 bits)
  e = 111        (3 bits)

Compression: Frequent symbols get shorter codes.
Fixed-length would need 3 bits each: 100 * 3 = 300 bits
Huffman: 45*1 + 12*3 + 13*3 + 5*4 + 9*4 + 16*3 = 224 bits (25% savings)
```

### 3. Fractional Knapsack

Unlike 0/1 knapsack, items can be divided. Take items by decreasing value-to-weight
ratio, taking fractions of the last item if needed.

```
FUNCTION fractional_knapsack(items: Array of (weight, value), capacity: Integer) -> Float:
    // Compute value-to-weight ratio for each item
    FOR EACH item IN items DO
        SET item.ratio = item.value / item.weight
    END FOR

    // Sort by ratio in descending order
    SORT items BY ratio DESCENDING

    SET total_value = 0.0
    SET remaining = capacity

    FOR EACH item IN items DO
        IF remaining = 0 THEN
            BREAK
        END IF
        IF item.weight <= remaining THEN
            // Take entire item
            SET total_value = total_value + item.value
            SET remaining = remaining - item.weight
        ELSE
            // Take fraction of item
            SET fraction = remaining / item.weight
            SET total_value = total_value + item.value * fraction
            SET remaining = 0
        END IF
    END FOR

    RETURN total_value
END FUNCTION
```

**Step-by-step trace for capacity=50, items: (10,60), (20,100), (30,120):**

```
Compute ratios:
  Item A: weight=10, value=60,  ratio=6.0
  Item B: weight=20, value=100, ratio=5.0
  Item C: weight=30, value=120, ratio=4.0

Sorted by ratio: [A(6.0), B(5.0), C(4.0)]

Greedy selection:
  Take A entirely: value += 60,  remaining = 50-10 = 40
  Take B entirely: value += 100, remaining = 40-20 = 20
  Take 2/3 of C:   value += 120 * (20/30) = 80, remaining = 0

Total value: 60 + 100 + 80 = 240

Visualization:
  |<--- A --->|<----- B ----->|<--- 2/3 of C --->|
  |    60     |     100       |        80         |
  0          10              30                   50
                                            capacity limit
```

### 4. Greedy Job Scheduling (Minimize Lateness)

Schedule jobs on a single machine to minimize maximum lateness, where each job has
a processing time and a deadline.

```
FUNCTION schedule_jobs(jobs: Array of (processing_time, deadline)) -> Array:
    // Sort by deadline (earliest deadline first)
    SORT jobs BY deadline ASCENDING

    SET schedule = EMPTY LIST
    SET current_time = 0

    FOR EACH job IN jobs DO
        SET job.start = current_time
        SET job.finish = current_time + job.processing_time
        SET job.lateness = MAX(0, job.finish - job.deadline)
        ADD job TO schedule
        SET current_time = job.finish
    END FOR

    RETURN schedule
END FUNCTION
```

**Step-by-step trace:**

```
Jobs: A(t=3,d=6), B(t=2,d=8), C(t=1,d=9), D(t=4,d=9), E(t=3,d=14), F(t=2,d=15)

Sorted by deadline: A, B, C, D, E, F

Schedule:
  Time:  0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15
         |--A--|--A--|--A--|
                           |--B--|--B--|
                                       |-C-|
                                          |---D---|---D---|
                                                          |--E--|--E--|--E--|
                                                                            |--F--|--F--|
  Job    Start  Finish  Deadline  Lateness
  A      0      3       6         0
  B      3      5       8         0
  C      5      6       9         0
  D      6      10      9         1  <-- maximum lateness
  E      10     13      14        0
  F      13     15      15        0

Maximum lateness: 1 (optimal -- no schedule can do better)
```

## Visual Walkthrough

### Greedy Choice in Activity Selection

```
Why "earliest finish time" works:

Consider two competing strategies for the first activity:

Strategy A: Pick earliest finish time (greedy)
  |---A---|
           |------remaining time to schedule more activities----->

Strategy B: Pick some other activity
  |--------B-----------|
                        |--less remaining time-->

Picking A leaves more room for future activities.
If any optimal solution does not include A, we can swap in A
(it finishes earlier) without reducing the total count.
This exchange argument proves the greedy choice property.
```

### Huffman Tree Construction Visual

```
Step-by-step tree building:

Initial:  [5] [9] [12] [13] [16] [45]

Step 1:   [14]  [12] [13] [16] [45]
          / \
        [5] [9]

Step 2:   [14] [25]  [16] [45]
          / \   / \
        [5][9][12][13]

Step 3:     [30]   [25] [45]
           / \      / \
        [14] [16] [12][13]
        / \
      [5] [9]

Step 4:       [55]        [45]
             / \
          [25] [30]
          / \   / \
       [12][13][14][16]
                / \
              [5] [9]

Step 5:         [100]
               / \
            [45] [55]
                  / \
              [25]  [30]
              / \    / \
           [12][13][14][16]
                    / \
                  [5] [9]
```

## Complexity Analysis

| Algorithm              | Time         | Space  | Correctness        |
|-----------------------|--------------|--------|--------------------|
| Activity Selection    | O(n log n)   | O(n)   | Always optimal     |
| Huffman Coding        | O(n log n)   | O(n)   | Always optimal     |
| Fractional Knapsack   | O(n log n)   | O(1)   | Always optimal     |
| 0/1 Knapsack (greedy) | O(n log n)   | O(1)   | NOT always optimal |
| Job Scheduling        | O(n log n)   | O(n)   | Always optimal     |
| Coin Change (greedy)  | O(n log n)   | O(1)   | NOT always optimal |
| Dijkstra (non-neg)    | O(E log V)   | O(V)   | Always optimal     |

Note: The O(n log n) time for most greedy algorithms comes from the sorting step.
The greedy selection itself is typically O(n).

### Greedy vs DP Performance Comparison

```
Problem: Knapsack with n items and capacity W

0/1 Knapsack:
  Greedy:  O(n log n) time, but may give WRONG answer
  DP:      O(n * W) time, always gives CORRECT answer

Fractional Knapsack:
  Greedy:  O(n log n) time, always gives CORRECT answer
  DP:      O(n * W) time, unnecessarily complex

The key: Fractional knapsack has the greedy choice property.
         0/1 knapsack does not.
```

## When to Use Greedy Algorithms

**Use greedy when:**
- The problem has both the greedy choice property and optimal substructure
- You can prove that the greedy choice leads to an optimal solution
- A fast approximation is acceptable (even when not provably optimal)
- The problem involves scheduling, selection, or compression tasks

**Do NOT use greedy when:**
- The problem requires considering combinations of choices (use DP)
- A local best choice can block a globally better option
- You cannot prove the greedy choice property holds
- Exact optimal solutions are required and greedy correctness is unproven

### Decision Flowchart

```
Does the problem have optimal substructure?
  |
  +-- No --> Neither Greedy nor DP works. Try other approaches.
  |
  +-- Yes --> Does it have overlapping subproblems?
                |
                +-- Yes --> Does the greedy choice property hold?
                |             |
                |             +-- Yes --> Use Greedy (fastest)
                |             +-- No  --> Use Dynamic Programming
                |
                +-- No --> Use Divide and Conquer
```

## Real-World Applications

- **Data compression:** Huffman coding is used in file compression formats and
  as a component of more advanced compression algorithms.

- **Network routing:** Dijkstra's shortest-path algorithm uses greedy selection
  of the next closest unvisited node.

- **Task scheduling:** Operating system schedulers use greedy algorithms to
  minimize lateness, maximize throughput, or balance loads.

- **Minimum spanning trees:** Kruskal's and Prim's algorithms (covered in the
  graph algorithms chapter) are greedy algorithms for connecting networks.

- **Change-making:** For standard currency denominations (where coins are
  powers or specific ratios), the greedy approach gives optimal change.

- **Caching strategies:** Least Recently Used (LRU) and other cache eviction
  policies use greedy heuristics to decide which items to keep.

## Common Pitfalls and Best Practices

1. **Assuming greedy works without proof.** Many problems look like they have the
   greedy choice property but do not. Always verify with a proof or counterexample.

2. **Confusing fractional and 0/1 problems.** Greedy works for fractional knapsack
   but fails for 0/1 knapsack. The ability to take fractions changes the problem
   fundamentally.

3. **Wrong greedy criterion.** Activity selection by shortest duration or by start
   time does not work. The criterion matters -- earliest finish time is correct.

4. **Not sorting first.** Most greedy algorithms require a sorted ordering.
   Forgetting the sort step or using the wrong comparison leads to incorrect results.

5. **Greedy for approximation.** Even when greedy does not give optimal results, it
   often gives a good approximation. Know when approximate solutions are acceptable.

6. **Overlooking edge cases.** Empty inputs, single-element inputs, and ties in the
   greedy criterion need careful handling.

## Practice Problems

1. Prove that the greedy activity selection algorithm produces an optimal solution
   using an exchange argument.

2. Build a Huffman tree for the following frequencies: E:15, T:12, A:10, O:8, N:7,
   R:6, I:5, S:4, H:3. What is the total encoded length?

3. Given coins of denominations [1, 5, 10, 25, 50], show that the greedy algorithm
   always gives optimal change for any amount. Then find a denomination set where
   greedy fails.

4. Solve the fractional knapsack problem for capacity=60 with items: (5,30), (10,40),
   (15,45), (22,77), (25,90). Show the greedy selection process.

5. Design a greedy algorithm for the interval coloring problem: given n intervals,
   find the minimum number of "colors" (resources) such that overlapping intervals
   receive different colors.

6. Compare greedy and DP solutions for the coin change problem with coins [1, 3, 4]
   and amount 6. Show why greedy fails and DP succeeds.

7. Implement a greedy algorithm for the minimum number of platforms needed at a
   train station given arrival and departure times of trains.

8. Prove that Huffman coding produces an optimal prefix-free code. Hint: use the
   fact that in an optimal code, the two least frequent symbols have the longest
   and equal-length codewords.
