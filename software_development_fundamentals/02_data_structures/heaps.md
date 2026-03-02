# Heaps

## Overview

A heap is a specialized tree-based data structure that satisfies the **heap
property**: in a **min-heap**, every parent node has a value less than or equal to
its children; in a **max-heap**, every parent has a value greater than or equal to
its children. This property guarantees that the minimum (or maximum) element is
always at the root, enabling O(1) access to the extremal element.

Heaps are most commonly implemented as **binary heaps** -- complete binary trees
stored compactly in an array. A complete binary tree fills every level except
possibly the last, which is filled from left to right. This regularity means no
pointers are needed; parent-child relationships are computed via simple index
arithmetic. This array representation gives heaps excellent cache performance and
minimal memory overhead.

The primary application of heaps is the **priority queue**, an abstract data type
where elements are served by priority rather than insertion order. Heaps also form
the basis of **heap sort**, an in-place O(n log n) comparison sort. They appear in
graph algorithms (Dijkstra's shortest path, Prim's minimum spanning tree), in median
maintenance, and in scheduling systems where the most urgent task must always be
identified quickly.

## Key Concepts

### The Heap Property

- **Min-Heap**: For every node i, `parent(i).value <= i.value`. The root holds the
  minimum value.
- **Max-Heap**: For every node i, `parent(i).value >= i.value`. The root holds the
  maximum value.

Note: A heap is NOT a sorted structure. Siblings have no ordering relationship. Only
the parent-child relationship is constrained.

### Complete Binary Tree

A complete binary tree has all levels fully filled except possibly the last, which
is filled strictly from left to right. This structure guarantees:
- Height is always `floor(log2(n))`.
- No gaps in the array representation.
- Efficient memory usage (no wasted slots).

### Array Representation

A binary heap stored in a zero-indexed array uses these index formulas:

```
Parent of node i:      floor((i - 1) / 2)
Left child of node i:  2 * i + 1
Right child of node i: 2 * i + 2
```

For one-indexed arrays:

```
Parent of node i:      floor(i / 2)
Left child of node i:  2 * i
Right child of node i: 2 * i + 1
```

### Heapify Operations

- **Sift Up (Bubble Up)**: After insertion at the end, repeatedly swap the new
  element with its parent until the heap property is restored. O(log n).
- **Sift Down (Bubble Down)**: After removing the root, move the replacement element
  down by swapping with the smaller (min-heap) or larger (max-heap) child until the
  heap property is restored. O(log n).

### Build Heap

Building a heap from an unsorted array can be done in O(n) time (not O(n log n))
using the **bottom-up heapify** approach: sift down each non-leaf node from the
last internal node up to the root. The mathematical proof shows that the sum of
work across all levels is bounded by O(n).

## Visual Representation

### Min-Heap as a Tree

```
              1
            /   \
           3     5
          / \   / \
         7   4 8   6
        / \
       9  10

Heap property: every parent <= its children
Root (1) is the minimum element
```

### Min-Heap as an Array

```
Index:   0    1    2    3    4    5    6    7    8
       +----+----+----+----+----+----+----+----+----+
Value: |  1 |  3 |  5 |  7 |  4 |  8 |  6 |  9 | 10 |
       +----+----+----+----+----+----+----+----+----+

Tree mapping:
  Index 0 (value 1) -> children at indices 1 (3) and 2 (5)
  Index 1 (value 3) -> children at indices 3 (7) and 4 (4)
  Index 2 (value 5) -> children at indices 5 (8) and 6 (6)
  Index 3 (value 7) -> children at indices 7 (9) and 8 (10)

Parent of index 4: floor((4-1)/2) = 1  (value 3)  -->  3 <= 4  (valid)
```

### Max-Heap

```
              50
            /    \
          30      40
         /  \    /  \
        10   20 15   35
       /
      5

Array: [50, 30, 40, 10, 20, 15, 35, 5]
Root (50) is the maximum element
```

### Insertion (Sift Up)

```
Insert 2 into the min-heap:

Step 1: Add at the end (index 9)
              1
            /   \
           3     5
          / \   / \
         7   4 8   6
        / \ /
       9 10 2

Step 2: Sift up -- compare 2 with parent 4 (index 4), swap
              1
            /   \
           3     5
          / \   / \
         7   2 8   6
        / \ /
       9 10 4

Step 3: Sift up -- compare 2 with parent 3 (index 1), swap
              1
            /   \
           2     5
          / \   / \
         7   3 8   6
        / \ /
       9 10 4

Step 4: Compare 2 with parent 1 (index 0), 2 > 1, stop.
```

### Deletion of Root (Extract Min)

```
Extract min from the heap:

Step 1: Save root (1), move last element (10) to root
              10
            /   \
           2     5
          / \   / \
         7   3 8   6
        / \
       9   4

Step 2: Sift down -- compare 10 with children 2 and 5, swap with 2
              2
            /   \
           10    5
          / \   / \
         7   3 8   6
        / \
       9   4

Step 3: Sift down -- compare 10 with children 7 and 3, swap with 3
              2
            /   \
           3     5
          / \   / \
         7  10  8   6
        / \
       9   4

Step 4: Sift down -- compare 10 with children 9 and 4, swap with 4
              2
            /   \
           3     5
          / \   / \
         7   4 8   6
        / \
       9  10

Done. Returned minimum: 1
```

## Operations and Pseudocode

### Core Heap Structure

```
STRUCTURE MinHeap:
    data: Array of Element
    count: Integer
    capacity: Integer
END STRUCTURE

FUNCTION createHeap(capacity: Integer) -> MinHeap:
    SET heap = NEW MinHeap
    SET heap.data = ALLOCATE Array of size capacity
    SET heap.count = 0
    SET heap.capacity = capacity
    RETURN heap
END FUNCTION
```

### Insert (Sift Up)

```
FUNCTION insert(heap: MinHeap, value: Element) -> Void:
    IF heap.count == heap.capacity THEN
        CALL resize(heap, heap.capacity * 2)
    END IF
    SET heap.data[heap.count] = value
    SET heap.count = heap.count + 1
    CALL siftUp(heap, heap.count - 1)
END FUNCTION

FUNCTION siftUp(heap: MinHeap, index: Integer) -> Void:
    WHILE index > 0 DO
        SET parentIndex = (index - 1) / 2
        IF heap.data[index] < heap.data[parentIndex] THEN
            SWAP heap.data[index] AND heap.data[parentIndex]
            SET index = parentIndex
        ELSE
            RETURN
        END IF
    END WHILE
END FUNCTION
```

### Extract Minimum (Sift Down)

```
FUNCTION extractMin(heap: MinHeap) -> Element:
    IF heap.count == 0 THEN
        ERROR "Heap is empty"
    END IF
    SET minValue = heap.data[0]
    SET heap.count = heap.count - 1
    SET heap.data[0] = heap.data[heap.count]
    CALL siftDown(heap, 0)
    RETURN minValue
END FUNCTION

FUNCTION siftDown(heap: MinHeap, index: Integer) -> Void:
    WHILE TRUE DO
        SET smallest = index
        SET left = 2 * index + 1
        SET right = 2 * index + 2

        IF left < heap.count AND heap.data[left] < heap.data[smallest] THEN
            SET smallest = left
        END IF
        IF right < heap.count AND heap.data[right] < heap.data[smallest] THEN
            SET smallest = right
        END IF

        IF smallest != index THEN
            SWAP heap.data[index] AND heap.data[smallest]
            SET index = smallest
        ELSE
            RETURN
        END IF
    END WHILE
END FUNCTION
```

### Peek (Get Minimum without Removal)

```
FUNCTION peekMin(heap: MinHeap) -> Element:
    IF heap.count == 0 THEN
        ERROR "Heap is empty"
    END IF
    RETURN heap.data[0]
END FUNCTION
```

### Build Heap (Bottom-Up, O(n))

```
FUNCTION buildHeap(array: Array, length: Integer) -> MinHeap:
    SET heap = NEW MinHeap
    SET heap.data = array
    SET heap.count = length
    SET heap.capacity = length

    // Start from last internal node, sift down each
    SET lastInternal = (length / 2) - 1
    FOR i FROM lastInternal DOWN TO 0 DO
        CALL siftDown(heap, i)
    END FOR

    RETURN heap
END FUNCTION
```

### Why Build Heap is O(n), not O(n log n)

```
For a heap of height h:
  - Level 0 (root):    1 node,  sifts down at most h levels
  - Level 1:           2 nodes, sifts down at most h-1 levels
  - Level k:           2^k nodes, sifts down at most h-k levels
  - Level h-1 (leaves' parents): 2^(h-1) nodes, sifts down at most 1 level
  - Level h (leaves):  not processed (no children)

Total work = SUM(k=0 to h-1) of 2^k * (h - k)
           = O(n)    (by geometric series analysis)

Intuition: Most nodes are near the bottom and sift down very little.
```

### Heap Sort

```
FUNCTION heapSort(array: Array, length: Integer) -> Void:
    // Phase 1: Build a max-heap in-place
    FOR i FROM (length / 2) - 1 DOWN TO 0 DO
        CALL maxSiftDown(array, length, i)
    END FOR

    // Phase 2: Repeatedly extract max and place at end
    FOR i FROM length - 1 DOWN TO 1 DO
        SWAP array[0] AND array[i]        // move max to sorted position
        CALL maxSiftDown(array, i, 0)     // restore heap on reduced array
    END FOR
END FUNCTION

FUNCTION maxSiftDown(array: Array, heapSize: Integer, index: Integer) -> Void:
    WHILE TRUE DO
        SET largest = index
        SET left = 2 * index + 1
        SET right = 2 * index + 2

        IF left < heapSize AND array[left] > array[largest] THEN
            SET largest = left
        END IF
        IF right < heapSize AND array[right] > array[largest] THEN
            SET largest = right
        END IF

        IF largest != index THEN
            SWAP array[index] AND array[largest]
            SET index = largest
        ELSE
            RETURN
        END IF
    END WHILE
END FUNCTION
```

### Heap Sort Trace

```
Array: [4, 10, 3, 5, 1]

Phase 1 - Build max-heap:
  [4, 10, 3, 5, 1] -> sift(1): [4, 10, 3, 5, 1] (no change)
  [4, 10, 3, 5, 1] -> sift(0): [10, 5, 3, 4, 1]

Phase 2 - Extract max repeatedly:
  Swap 10 and 1:  [1, 5, 3, 4, | 10]    sift -> [5, 4, 3, 1, | 10]
  Swap 5 and 1:   [1, 4, 3, | 5, 10]    sift -> [4, 1, 3, | 5, 10]
  Swap 4 and 3:   [3, 1, | 4, 5, 10]    sift -> [3, 1, | 4, 5, 10]
  Swap 3 and 1:   [1, | 3, 4, 5, 10]    done

Sorted: [1, 3, 4, 5, 10]
```

### Priority Queue Using Min-Heap

```
FUNCTION priorityQueueInsert(pq: MinHeap, element: Element, priority: Number) -> Void:
    SET entry = NEW PriorityEntry(priority, element)
    CALL insert(pq, entry)
END FUNCTION

FUNCTION priorityQueueExtract(pq: MinHeap) -> Element:
    SET entry = CALL extractMin(pq)
    RETURN entry.element
END FUNCTION

FUNCTION decreaseKey(heap: MinHeap, index: Integer, newPriority: Number) -> Void:
    IF newPriority > heap.data[index].priority THEN
        ERROR "New priority is larger than current"
    END IF
    SET heap.data[index].priority = newPriority
    CALL siftUp(heap, index)
END FUNCTION
```

## Complexity Analysis

| Operation       | Min/Max Heap | Notes                              |
|-----------------|-------------|------------------------------------|
| Peek min/max    | O(1)        | Root is always the extremal value  |
| Insert          | O(log n)    | Sift up from bottom to root        |
| Extract min/max | O(log n)    | Sift down from root to bottom      |
| Build heap      | O(n)        | Bottom-up heapify                  |
| Decrease key    | O(log n)    | Sift up after priority change      |
| Delete arbitrary| O(log n)    | Decrease key to min, then extract  |
| Search          | O(n)        | Heap is not optimized for search   |
| Merge two heaps | O(n)        | Build heap on combined array       |
| Space           | O(n)        | Array-based, no pointer overhead   |

### Heap Sort Complexity

| Metric       | Value     | Notes                                |
|--------------|-----------|--------------------------------------|
| Best case    | O(n log n)| Always performs full sort             |
| Average case | O(n log n)| Consistent performance               |
| Worst case   | O(n log n)| Guaranteed, unlike quicksort         |
| Space        | O(1)      | In-place (no auxiliary array needed)  |
| Stable       | No        | Relative order of equals not kept    |

## When to Use / When NOT to Use

### Use Heaps When:
- You need fast access to the minimum or maximum element.
- Implementing a priority queue.
- You need O(n log n) guaranteed in-place sorting (heap sort).
- Running Dijkstra's or Prim's algorithm.
- Maintaining a running median (use two heaps: max-heap and min-heap).
- Merging k sorted lists (use a min-heap of size k).

### Avoid Heaps When:
- You need fast search for arbitrary elements (use a BST or hash table).
- You need sorted traversal of all elements (use a BST).
- You need both min and max access (use a min-max heap or two heaps).
- The data is already sorted (a sorted array gives O(1) min/max access).
- You need stable sorting (heap sort is not stable).

## Real-World Applications

- **Priority queues**: Operating system task schedulers, interrupt handling.
- **Dijkstra's algorithm**: Shortest path in weighted graphs using a min-heap.
- **Prim's algorithm**: Minimum spanning tree using a min-heap of edge weights.
- **Heap sort**: In-place O(n log n) sort with guaranteed worst-case performance.
- **Median maintenance**: Two heaps (max-heap for lower half, min-heap for upper
  half) to find the running median in O(log n) per insertion.
- **K-way merge**: Merging k sorted streams using a min-heap of size k.
- **Top-K elements**: Finding the k largest elements in a stream using a min-heap
  of size k.
- **Event simulation**: Processing events in timestamp order.
- **Huffman coding**: Building optimal prefix codes using a priority queue.

## Common Pitfalls and Best Practices

1. **Confusing min-heap and max-heap**: Ensure the comparison direction matches the
   heap type. Mixing them corrupts the heap property.
2. **Off-by-one in index calculations**: Zero-indexed and one-indexed formulas are
   different. Pick one and use it consistently.
3. **Forgetting to sift after modification**: Every insert needs sift-up, every
   extract needs sift-down. Forgetting either breaks the heap property.
4. **Using heaps for search**: Heaps are not designed for O(log n) search. If you
   need to find arbitrary elements, augment with a hash table (index map).
5. **Assuming heaps are sorted**: A valid min-heap [1, 5, 2, 7, 6, 3, 4] is NOT
   sorted. Only the root-to-leaf paths are partially ordered.
6. **Inefficient heap construction**: Building a heap by repeated insertion is
   O(n log n). Use the O(n) bottom-up buildHeap instead.
7. **Not handling empty heap**: Always check for empty heap before extract or peek.

## Comparison with Related Structures

| Feature              | Binary Heap  | Sorted Array | BST (balanced)| Fibonacci Heap |
|----------------------|--------------|--------------|---------------|----------------|
| Find min/max         | O(1)         | O(1)         | O(log n)      | O(1)           |
| Insert               | O(log n)     | O(n)         | O(log n)      | O(1)*          |
| Extract min/max      | O(log n)     | O(1) or O(n) | O(log n)      | O(log n)*      |
| Decrease key         | O(log n)     | O(n)         | O(log n)      | O(1)*          |
| Merge                | O(n)         | O(n)         | O(n)          | O(1)*          |
| Search               | O(n)         | O(log n)     | O(log n)      | O(n)           |
| Build from array     | O(n)         | O(n log n)   | O(n log n)    | O(n)           |
| Space overhead       | None (array) | None (array) | Pointers      | Pointers       |
| Implementation       | Simple       | Simple       | Moderate      | Complex        |

\* Amortized time complexity.

## Practice Problems

1. **Kth Largest Element**: Find the kth largest element in an unsorted array. Use
   a min-heap of size k for O(n log k) time, or use quickselect for O(n) average.

2. **Merge K Sorted Lists**: Given k sorted lists, merge them into one sorted list
   using a min-heap. Achieve O(N log k) time where N is total elements.

3. **Find Median in a Stream**: Design a structure that supports adding numbers and
   finding the current median. Use a max-heap and min-heap together.

4. **Top K Frequent Elements**: Given an array, find the k most frequent elements.
   Use a hash table for counting and a min-heap of size k.

5. **Heap Sort Implementation**: Implement heap sort from scratch using an in-place
   max-heap. Verify it handles duplicates, already-sorted, and reverse-sorted input.

6. **Task Scheduler**: Given tasks with deadlines and profits, schedule tasks to
   maximize total profit using a max-heap and greedy algorithm.
