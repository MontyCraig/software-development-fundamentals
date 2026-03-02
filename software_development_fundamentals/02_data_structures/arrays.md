# Arrays

## Overview

An array is the most fundamental data structure in computing. It stores a fixed-size
(or dynamically resizable) sequence of elements of the same type in contiguous memory
locations. Because elements are laid out adjacently in memory, any element can be
accessed in constant time by computing an offset from the base address.

Arrays come in two primary flavors. A **static array** has a size determined at
creation time and cannot grow or shrink. A **dynamic array** (sometimes called a
resizable array or vector) wraps a static array and automatically allocates a larger
backing store when the current one fills up. This resizing strategy gives dynamic
arrays an amortized constant-time append, making them suitable for collections whose
final size is unknown in advance.

Understanding arrays is essential because virtually every other data structure is
either built on top of arrays or compared against them. Hash tables use arrays of
buckets, heaps use arrays as their backing store, and even graph adjacency lists are
arrays of arrays.

## Key Concepts

### Contiguous Memory Layout

Every element in an array occupies a predictable position in memory. Given a base
address `B`, an element size `S`, and an index `i`, the address of element `i` is:

```
address(i) = B + (i * S)
```

This arithmetic is what makes index-based access O(1).

### Zero-Based vs One-Based Indexing

Most implementations use zero-based indexing, where the first element is at index 0
and the last is at index `length - 1`. Some systems use one-based indexing. The
choice is a convention; the underlying math adjusts accordingly.

### Static Arrays

A static array is allocated once with a fixed capacity. It cannot grow. If you need
more space, you must allocate a new, larger array and copy elements over.

### Dynamic Arrays and Amortized Resizing

A dynamic array maintains three values: the backing static array, the current
**count** (number of elements stored), and the **capacity** (total slots available).
When count equals capacity and a new element must be added, the array:

1. Allocates a new array of `capacity * GROWTH_FACTOR` (commonly 2).
2. Copies all existing elements to the new array.
3. Releases the old array.
4. Inserts the new element.

Although a single resize costs O(n), it happens infrequently enough that the
**amortized** cost per append is O(1).

### Multidimensional Arrays

A two-dimensional array (matrix) can be stored in **row-major** order (all elements
of row 0, then row 1, ...) or **column-major** order. The address formula generalizes:

```
address(row, col) = B + (row * NUM_COLS + col) * S      // row-major
address(row, col) = B + (col * NUM_ROWS + row) * S      // column-major
```

## Visual Representation

### Static Array in Memory

```
Index:    0       1       2       3       4
        +-------+-------+-------+-------+-------+
Value:  |  42   |  17   |  93   |   8   |  61   |
        +-------+-------+-------+-------+-------+
Address: 0x100   0x104   0x108   0x10C   0x110

Base address = 0x100, Element size = 4 bytes
address(3) = 0x100 + (3 * 4) = 0x10C  -->  value 8
```

### Dynamic Array Resize

```
BEFORE (capacity=4, count=4):        AFTER (capacity=8, count=5):
+---+---+---+---+                    +---+---+---+---+---+---+---+---+
| A | B | C | D |                    | A | B | C | D | E |   |   |   |
+---+---+---+---+                    +---+---+---+---+---+---+---+---+
  count=4  cap=4                       count=5         cap=8

Step 1: Allocate new array of size 8
Step 2: Copy A, B, C, D to new array
Step 3: Insert E at index 4
Step 4: Free old array
```

### Two-Dimensional Array (Row-Major)

```
Logical view:           Memory layout (row-major):
+---+---+---+           +---+---+---+---+---+---+---+---+---+
| 1 | 2 | 3 |  row 0   | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
+---+---+---+           +---+---+---+---+---+---+---+---+---+
| 4 | 5 | 6 |  row 1     row 0       row 1       row 2
+---+---+---+
| 7 | 8 | 9 |  row 2
+---+---+---+
```

## Operations and Pseudocode

### Access by Index

```
FUNCTION get(array: Array, index: Integer) -> Element:
    IF index < 0 OR index >= array.count THEN
        ERROR "Index out of bounds"
    END IF
    RETURN array.data[index]
END FUNCTION
```

### Update by Index

```
FUNCTION set(array: Array, index: Integer, value: Element) -> Void:
    IF index < 0 OR index >= array.count THEN
        ERROR "Index out of bounds"
    END IF
    SET array.data[index] = value
END FUNCTION
```

### Append (Dynamic Array)

```
FUNCTION append(array: DynamicArray, value: Element) -> Void:
    IF array.count == array.capacity THEN
        CALL resize(array, array.capacity * 2)
    END IF
    SET array.data[array.count] = value
    SET array.count = array.count + 1
END FUNCTION
```

### Resize (Dynamic Array)

```
FUNCTION resize(array: DynamicArray, newCapacity: Integer) -> Void:
    SET newData = ALLOCATE Array of size newCapacity
    FOR i FROM 0 TO array.count - 1 DO
        SET newData[i] = array.data[i]
    END FOR
    FREE array.data
    SET array.data = newData
    SET array.capacity = newCapacity
END FUNCTION
```

### Insert at Position

```
FUNCTION insertAt(array: DynamicArray, index: Integer, value: Element) -> Void:
    IF index < 0 OR index > array.count THEN
        ERROR "Index out of bounds"
    END IF
    IF array.count == array.capacity THEN
        CALL resize(array, array.capacity * 2)
    END IF
    FOR i FROM array.count DOWN TO index + 1 DO
        SET array.data[i] = array.data[i - 1]
    END FOR
    SET array.data[index] = value
    SET array.count = array.count + 1
END FUNCTION
```

### Delete at Position

```
FUNCTION deleteAt(array: DynamicArray, index: Integer) -> Element:
    IF index < 0 OR index >= array.count THEN
        ERROR "Index out of bounds"
    END IF
    SET removed = array.data[index]
    FOR i FROM index TO array.count - 2 DO
        SET array.data[i] = array.data[i + 1]
    END FOR
    SET array.count = array.count - 1
    RETURN removed
END FUNCTION
```

### Linear Search

```
FUNCTION linearSearch(array: Array, target: Element) -> Integer:
    FOR i FROM 0 TO array.count - 1 DO
        IF array.data[i] == target THEN
            RETURN i
        END IF
    END FOR
    RETURN -1
END FUNCTION
```

### Binary Search (Sorted Array)

```
FUNCTION binarySearch(array: Array, target: Element) -> Integer:
    SET low = 0
    SET high = array.count - 1
    WHILE low <= high DO
        SET mid = low + (high - low) / 2
        IF array.data[mid] == target THEN
            RETURN mid
        ELSE IF array.data[mid] < target THEN
            SET low = mid + 1
        ELSE
            SET high = mid - 1
        END IF
    END WHILE
    RETURN -1
END FUNCTION
```

## Complexity Analysis

| Operation         | Best Case | Average Case | Worst Case | Space   |
|-------------------|-----------|--------------|------------|---------|
| Access by index   | O(1)      | O(1)         | O(1)       | O(1)    |
| Update by index   | O(1)      | O(1)         | O(1)       | O(1)    |
| Append (dynamic)  | O(1)      | O(1)*        | O(n)       | O(n)    |
| Insert at start   | O(n)      | O(n)         | O(n)       | O(1)    |
| Insert at middle  | O(n)      | O(n)         | O(n)       | O(1)    |
| Delete at start   | O(n)      | O(n)         | O(n)       | O(1)    |
| Delete at end     | O(1)      | O(1)         | O(1)       | O(1)    |
| Linear search     | O(1)      | O(n)         | O(n)       | O(1)    |
| Binary search     | O(1)      | O(log n)     | O(log n)   | O(1)    |
| Resize            | O(n)      | O(n)         | O(n)       | O(n)    |

\* Amortized O(1) -- individual appends may cost O(n) during resize, but across
a sequence of n appends, the total cost is O(n), yielding O(1) per operation.

### Amortized Analysis of Dynamic Array Append

Using the **accounting method**: charge each append 3 units of work.
- 1 unit to insert the element.
- 1 unit to later copy this element during a resize.
- 1 unit to copy one previously existing element during the same resize.

Since each element pays for its own future copy and helps pay for one old element,
the total cost of n appends is 3n = O(n), giving O(1) amortized per append.

## When to Use / When NOT to Use

### Use Arrays When:
- You need fast random access by index (O(1) lookups).
- The collection size is known in advance (static array) or grows primarily at
  the end (dynamic array).
- Memory locality matters -- arrays are cache-friendly because elements are
  contiguous.
- You need a backing store for other structures (heaps, hash tables).

### Avoid Arrays When:
- You need frequent insertions or deletions at arbitrary positions (O(n) shifting).
- The size fluctuates dramatically and you cannot tolerate occasional O(n) resizes.
- You need efficient insertion at the front (use a deque or linked list instead).
- Elements vary widely in size (arrays expect uniform element size).

## Real-World Applications

- **Image pixels**: A 2D array of color values.
- **Spreadsheet cells**: Rows and columns stored as a 2D array.
- **Audio samples**: A 1D array of amplitude values at fixed sample intervals.
- **Lookup tables**: Precomputed results indexed by input value.
- **Buffers**: I/O buffers, ring buffers for streaming data.
- **Database rows**: Fixed-schema rows stored as arrays of fields.

## Common Pitfalls and Best Practices

1. **Off-by-one errors**: The most common bug. Remember that a zero-indexed array
   of length n has valid indices 0 through n-1.
2. **Forgetting bounds checks**: Always validate indices before access.
3. **Choosing a poor growth factor**: A factor of 2 is standard. A factor of 1
   (adding a fixed amount) degrades append to O(n) amortized. A very large factor
   wastes memory.
4. **Not shrinking**: If a dynamic array drops to 25% utilization, consider halving
   its capacity to reclaim memory.
5. **Ignoring cache effects**: Accessing array elements sequentially (stride-1) is
   far faster than random jumps due to CPU cache lines.
6. **Aliasing**: When two references point to the same array, modifications through
   one are visible through the other. Copy when independent mutation is needed.

## Comparison with Related Structures

| Feature              | Static Array | Dynamic Array | Linked List | Deque      |
|----------------------|--------------|---------------|-------------|------------|
| Random access        | O(1)         | O(1)          | O(n)        | O(1)       |
| Append               | N/A (fixed)  | O(1)*         | O(1)        | O(1)*      |
| Prepend              | O(n)         | O(n)          | O(1)        | O(1)*      |
| Insert at middle     | O(n)         | O(n)          | O(1)**      | O(n)       |
| Delete at middle     | O(n)         | O(n)          | O(1)**      | O(n)       |
| Memory overhead      | None         | Unused slots   | Per-node ptr| Unused slots|
| Cache friendliness   | Excellent    | Excellent     | Poor        | Good       |

\* Amortized
\** O(1) after reaching the node; finding the node is O(n)

## Practice Problems

1. **Rotate Array**: Given an array of n elements, rotate it right by k positions.
   Achieve O(n) time and O(1) extra space using the reversal algorithm.

2. **Merge Sorted Arrays**: Given two sorted arrays, produce a single sorted array
   in O(n + m) time.

3. **Maximum Subarray Sum**: Find the contiguous subarray with the largest sum
   (Kadane's algorithm, O(n) time).

4. **Two Sum**: Given an array and a target sum, find two elements that add up to
   the target. Solve in O(n) time using a hash table, or O(n log n) with sorting.

5. **Dynamic Array from Scratch**: Implement a dynamic array supporting append,
   insertAt, deleteAt, and get, with automatic resizing at a growth factor of 2.

6. **Sparse Array**: Design a structure that behaves like an array but efficiently
   handles cases where most elements are a default value (e.g., zero). Compare the
   trade-offs with a standard dense array.
