# Comparison Sorting Algorithms

## Overview

Comparison sorting algorithms order elements by repeatedly comparing pairs and deciding
their relative order. This family includes some of the most fundamental algorithms in
computer science, ranging from simple quadratic-time approaches ideal for small or
nearly-sorted data to sophisticated divide-and-conquer methods that achieve the
theoretically optimal O(n log n) bound.

Every comparison sort must make at least O(n log n) comparisons in the worst case. This
lower bound arises from the decision tree model: with n! possible permutations and each
comparison producing a binary outcome, the tree height must be at least log2(n!), which
is Theta(n log n). Understanding this bound helps explain why no comparison sort can do
better asymptotically.

A key property distinguishing sorts is stability: whether equal elements retain their
original relative order. Stability matters when sorting by multiple keys (sort by name,
then by age) or when the original order carries meaning. This chapter covers six
comparison sorts, from the elementary to the advanced.

## Key Concepts

### Stability

A sorting algorithm is stable if elements with equal keys appear in the output in the
same order as they appear in the input.

```
Input:  [(3,"a"), (1,"b"), (3,"c"), (1,"d")]
Stable sort by first element:   [(1,"b"), (1,"d"), (3,"a"), (3,"c")]
Unstable sort by first element: [(1,"d"), (1,"b"), (3,"c"), (3,"a")]  // possible
```

### In-Place vs Out-of-Place

An in-place sort uses O(1) additional memory beyond the input. An out-of-place sort
requires extra space proportional to the input size.

### Divide and Conquer

Several efficient sorts split the problem into subproblems, solve them recursively,
and combine results. This strategy yields O(n log n) performance.

## Algorithms and Pseudocode

### 1. Bubble Sort

Repeatedly swap adjacent elements that are out of order. After each pass, the largest
unsorted element "bubbles" to its correct position.

```
FUNCTION bubble_sort(A: Array of Comparable, n: Integer):
    FOR i = 0 TO n - 2 DO
        SET swapped = false
        FOR j = 0 TO n - 2 - i DO
            IF A[j] > A[j + 1] THEN
                SWAP(A[j], A[j + 1])
                SET swapped = true
            END IF
        END FOR
        IF NOT swapped THEN
            RETURN                          // early exit if already sorted
        END IF
    END FOR
END FUNCTION
```

**Step-by-step trace on [5, 3, 8, 1, 2]:**

```
Pass 1: [5,3,8,1,2] -> [3,5,8,1,2] -> [3,5,8,1,2] -> [3,5,1,8,2] -> [3,5,1,2,8]
Pass 2: [3,5,1,2,8] -> [3,5,1,2,8] -> [3,1,5,2,8] -> [3,1,2,5,8]
Pass 3: [3,1,2,5,8] -> [1,3,2,5,8] -> [1,2,3,5,8]
Pass 4: [1,2,3,5,8] -> no swaps -> DONE (early exit)
```

### 2. Selection Sort

Find the minimum element in the unsorted region and swap it to the front.

```
FUNCTION selection_sort(A: Array of Comparable, n: Integer):
    FOR i = 0 TO n - 2 DO
        SET min_index = i
        FOR j = i + 1 TO n - 1 DO
            IF A[j] < A[min_index] THEN
                SET min_index = j
            END IF
        END FOR
        IF min_index != i THEN
            SWAP(A[i], A[min_index])
        END IF
    END FOR
END FUNCTION
```

**Step-by-step trace on [29, 10, 14, 37, 13]:**

```
i=0: min_index=1 (value 10)  -> SWAP(29,10) -> [10, 29, 14, 37, 13]
i=1: min_index=4 (value 13)  -> SWAP(29,13) -> [10, 13, 14, 37, 29]
i=2: min_index=2 (value 14)  -> no swap     -> [10, 13, 14, 37, 29]
i=3: min_index=4 (value 29)  -> SWAP(37,29) -> [10, 13, 14, 29, 37]
Result: [10, 13, 14, 29, 37]
```

### 3. Insertion Sort

Build the sorted portion one element at a time by inserting each element into its
correct position among the already-sorted elements.

```
FUNCTION insertion_sort(A: Array of Comparable, n: Integer):
    FOR i = 1 TO n - 1 DO
        SET key = A[i]
        SET j = i - 1
        WHILE j >= 0 AND A[j] > key DO
            SET A[j + 1] = A[j]
            SET j = j - 1
        END WHILE
        SET A[j + 1] = key
    END FOR
END FUNCTION
```

**Step-by-step trace on [7, 4, 5, 2]:**

```
i=1: key=4, shift 7 right       -> [4, 7, 5, 2]
         sorted: [4, 7]
i=2: key=5, shift 7 right       -> [4, 5, 7, 2]
         sorted: [4, 5, 7]
i=3: key=2, shift 7,5,4 right   -> [2, 4, 5, 7]
         sorted: [2, 4, 5, 7]
```

### 4. Merge Sort

Divide the array in half, recursively sort each half, then merge the two sorted
halves into one sorted array.

```
FUNCTION merge_sort(A: Array of Comparable, left: Integer, right: Integer):
    IF left >= right THEN
        RETURN
    END IF
    SET mid = FLOOR((left + right) / 2)
    merge_sort(A, left, mid)
    merge_sort(A, mid + 1, right)
    merge(A, left, mid, right)
END FUNCTION

FUNCTION merge(A: Array, left: Integer, mid: Integer, right: Integer):
    SET L = COPY(A[left .. mid])
    SET R = COPY(A[mid + 1 .. right])
    SET i = 0, j = 0, k = left
    WHILE i < LENGTH(L) AND j < LENGTH(R) DO
        IF L[i] <= R[j] THEN               // <= preserves stability
            SET A[k] = L[i]
            SET i = i + 1
        ELSE
            SET A[k] = R[j]
            SET j = j + 1
        END IF
        SET k = k + 1
    END WHILE
    WHILE i < LENGTH(L) DO
        SET A[k] = L[i]
        SET i = i + 1
        SET k = k + 1
    END WHILE
    WHILE j < LENGTH(R) DO
        SET A[k] = R[j]
        SET j = j + 1
        SET k = k + 1
    END WHILE
END FUNCTION
```

**Step-by-step trace on [38, 27, 43, 3, 9, 82, 10]:**

```
Split:  [38, 27, 43, 3] | [9, 82, 10]
Split:  [38, 27] | [43, 3]    [9, 82] | [10]
Split:  [38]|[27]  [43]|[3]   [9]|[82]  [10]
Merge:  [27, 38]   [3, 43]    [9, 82]   [10]
Merge:  [3, 27, 38, 43]       [9, 10, 82]
Merge:  [3, 9, 10, 27, 38, 43, 82]
```

### 5. Quick Sort

Select a pivot, partition elements into those less than and greater than the pivot,
then recursively sort the partitions.

```
FUNCTION quick_sort(A: Array of Comparable, low: Integer, high: Integer):
    IF low < high THEN
        SET pivot_index = partition(A, low, high)
        quick_sort(A, low, pivot_index - 1)
        quick_sort(A, pivot_index + 1, high)
    END IF
END FUNCTION

FUNCTION partition(A: Array, low: Integer, high: Integer) -> Integer:
    SET pivot = A[high]                     // choose last element as pivot
    SET i = low - 1
    FOR j = low TO high - 1 DO
        IF A[j] <= pivot THEN
            SET i = i + 1
            SWAP(A[i], A[j])
        END IF
    END FOR
    SWAP(A[i + 1], A[high])
    RETURN i + 1
END FUNCTION
```

**Step-by-step trace on [10, 7, 8, 9, 1, 5], pivot = 5:**

```
Initial: [10, 7, 8, 9, 1, 5]  pivot=5, i=-1
j=0: 10>5, skip                           i=-1
j=1: 7>5, skip                            i=-1
j=2: 8>5, skip                            i=-1
j=3: 9>5, skip                            i=-1
j=4: 1<=5, i=0, SWAP(A[0],A[4])          [1, 7, 8, 9, 10, 5]
Final: SWAP(A[1],A[5])                    [1, 5, 8, 9, 10, 7]
Pivot 5 at index 1.
Recurse on [1] and [8, 9, 10, 7]
```

### 6. Heap Sort

Build a max-heap from the array, then repeatedly extract the maximum and place it
at the end.

```
FUNCTION heap_sort(A: Array of Comparable, n: Integer):
    // Build max-heap
    FOR i = FLOOR(n / 2) - 1 DOWNTO 0 DO
        heapify_down(A, n, i)
    END FOR
    // Extract elements
    FOR i = n - 1 DOWNTO 1 DO
        SWAP(A[0], A[i])
        heapify_down(A, i, 0)
    END FOR
END FUNCTION

FUNCTION heapify_down(A: Array, heap_size: Integer, i: Integer):
    SET largest = i
    SET left = 2 * i + 1
    SET right = 2 * i + 2
    IF left < heap_size AND A[left] > A[largest] THEN
        SET largest = left
    END IF
    IF right < heap_size AND A[right] > A[largest] THEN
        SET largest = right
    END IF
    IF largest != i THEN
        SWAP(A[i], A[largest])
        heapify_down(A, heap_size, largest)
    END IF
END FUNCTION
```

**Step-by-step trace on [4, 10, 3, 5, 1]:**

```
Build heap:
  Start:     [4, 10, 3, 5, 1]
  Heapify i=1: [4, 10, 3, 5, 1] (10 > 5,1 -- already valid)
  Heapify i=0: [10, 5, 3, 4, 1] (swap 4<->10, then 4<->5)
  Max-heap:  [10, 5, 3, 4, 1]

Extract:
  Swap 10<->1: [1, 5, 3, 4, | 10]  heapify -> [5, 4, 3, 1, | 10]
  Swap 5<->1:  [1, 4, 3, | 5, 10]  heapify -> [4, 1, 3, | 5, 10]
  Swap 4<->3:  [3, 1, | 4, 5, 10]  heapify -> [3, 1, | 4, 5, 10]
  Swap 3<->1:  [1, | 3, 4, 5, 10]
  Result:      [1, 3, 4, 5, 10]
```

## Visual Walkthrough

### Merge Sort Divide-and-Conquer Tree

```
                    [38, 27, 43, 3, 9, 82, 10]
                   /                           \
          [38, 27, 43, 3]                [9, 82, 10]
          /             \                /          \
      [38, 27]      [43, 3]        [9, 82]       [10]
      /     \       /     \        /     \          |
    [38]   [27]   [43]   [3]    [9]    [82]       [10]
      \     /       \     /      \     /            |
      [27, 38]      [3, 43]     [9, 82]          [10]
          \             /            \             /
      [3, 27, 38, 43]              [9, 10, 82]
                \                      /
          [3, 9, 10, 27, 38, 43, 82]
```

### Quick Sort Partition Visualization

```
Array: [10, 7, 8, 9, 1, 5]   Pivot = 5

Step: Compare each element to pivot

  [10, 7, 8, 9, 1, 5]
   ^                ^
   j               pivot
   10 > 5? YES -> skip

  [10, 7, 8, 9, 1, 5]
       ^
       7 > 5? YES -> skip

  ... (8, 9 also skip) ...

  [10, 7, 8, 9, 1, 5]
                ^
                1 <= 5? YES -> swap 1 with first "big" element

  [ 1, 7, 8, 9, 10, 5]
       ^             ^
       |             pivot goes here
       swap pivot into position

  [ 1, 5, 8, 9, 10, 7]
       ^
     pivot in final position

  Left partition: [1]     Right partition: [8, 9, 10, 7]
```

## Complexity Analysis

| Algorithm      | Best       | Average    | Worst      | Space   | Stable? |
|---------------|------------|------------|------------|---------|---------|
| Bubble Sort   | O(n)       | O(n^2)    | O(n^2)    | O(1)    | Yes     |
| Selection Sort| O(n^2)    | O(n^2)    | O(n^2)    | O(1)    | No      |
| Insertion Sort| O(n)       | O(n^2)    | O(n^2)    | O(1)    | Yes     |
| Merge Sort    | O(n log n) | O(n log n) | O(n log n) | O(n)   | Yes     |
| Quick Sort    | O(n log n) | O(n log n) | O(n^2)    | O(log n)| No     |
| Heap Sort     | O(n log n) | O(n log n) | O(n log n) | O(1)   | No      |

## When to Use Each Algorithm

- **Bubble Sort:** Educational purposes only. Useful for very small or nearly-sorted
  data where the early-exit optimization helps.

- **Selection Sort:** When the number of swaps must be minimized. Always performs
  exactly n-1 swaps regardless of input order.

- **Insertion Sort:** Small arrays (n < 20-50), nearly sorted data, or as the base
  case for hybrid sorts. Adaptive -- runs in O(n) on sorted input.

- **Merge Sort:** When guaranteed O(n log n) worst-case performance and stability
  are required. Ideal for linked lists (no random access needed, O(1) merge space).

- **Quick Sort:** General-purpose sorting for arrays. Fastest in practice due to
  cache-friendly sequential access and small constant factors. Use median-of-three
  pivot selection to avoid worst-case.

- **Heap Sort:** When O(n log n) worst-case is needed with O(1) extra space. Good
  for real-time systems where consistent performance matters.

## Real-World Applications

- **Database engines** use merge sort variants for external sorting of large datasets
  that do not fit in memory.
- **Standard library sorts** typically use hybrid approaches: introsort (quick sort
  with heap sort fallback) or timsort (merge sort with insertion sort for small runs).
- **Insertion sort** is used as the base case in nearly all production sorting
  implementations because it outperforms divide-and-conquer for small n.
- **Priority queues** use heap sort principles for scheduling tasks.

## Common Pitfalls and Best Practices

1. **Using bubble sort in production.** It is almost always the wrong choice for
   real applications. Prefer insertion sort for simple cases.

2. **Quick sort without pivot strategy.** Always-choosing the first or last element
   causes O(n^2) on sorted input. Use median-of-three or random pivots.

3. **Forgetting stability requirements.** If you need stability, avoid selection
   sort, heap sort, and basic quick sort.

4. **Ignoring cache effects.** Merge sort accesses memory sequentially but needs
   extra space. Quick sort is more cache-friendly due to in-place partitioning.

5. **Not using hybrid approaches.** Production sorts switch to insertion sort
   below a threshold (typically 16-32 elements) for optimal real-world performance.

## Practice Problems

1. Trace insertion sort on the array [12, 11, 13, 5, 6]. Show the array state
   after each insertion.

2. Implement a version of quick sort that uses median-of-three pivot selection.

3. Prove that selection sort always performs exactly n(n-1)/2 comparisons regardless
   of the input order.

4. Modify merge sort to count the number of inversions in an array. (Two elements
   A[i] and A[j] form an inversion if i < j and A[i] > A[j].)

5. Explain why quick sort is typically faster than merge sort in practice despite
   both having O(n log n) average-case complexity.

6. Design a hybrid sort that uses quick sort for large partitions and insertion sort
   for partitions below size 16. Describe the expected performance improvement.
