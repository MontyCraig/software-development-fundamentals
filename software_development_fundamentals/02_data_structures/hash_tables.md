# Hash Tables

## Overview

A hash table (also called a hash map or dictionary) is a data structure that provides
near-constant-time average-case performance for insertions, deletions, and lookups.
It achieves this by using a **hash function** to compute an index (called a **hash
code** or **bucket index**) into an array of slots, where the corresponding value is
stored or retrieved.

The core challenge of hash tables is handling **collisions** -- when two different
keys produce the same bucket index. Two primary strategies exist: **chaining** (each
bucket holds a linked list of entries) and **open addressing** (colliding entries
probe for the next available slot within the array itself). The choice between these
strategies involves trade-offs in memory usage, cache performance, and complexity.

Hash tables are among the most widely used data structures in practice. They power
dictionaries, sets, caches, database indexes, and symbol tables. Understanding hash
functions, collision resolution, load factors, and rehashing is essential for using
hash tables effectively and diagnosing performance problems when they arise.

## Key Concepts

### Hash Functions

A hash function maps a key to an integer index in the range [0, capacity - 1]:

```
index = hash(key) MOD capacity
```

A good hash function should:
- **Be deterministic**: The same key always produces the same hash.
- **Distribute uniformly**: Keys should spread evenly across buckets.
- **Be fast to compute**: Hashing should not be a bottleneck.
- **Minimize collisions**: Different keys should rarely map to the same index.

### Common Hash Techniques

- **Division method**: `hash(key) = key MOD capacity` (capacity should be prime).
- **Multiplication method**: `hash(key) = floor(capacity * frac(key * A))` where
  A is a constant (often the golden ratio inverse, 0.6180339887).
- **String hashing**: Process each character: `hash = hash * 31 + char_code`.
- **Universal hashing**: Randomly chosen hash function from a family, guaranteeing
  low expected collisions regardless of input distribution.

### Load Factor

The **load factor** (alpha) measures how full the table is:

```
alpha = number_of_entries / capacity
```

- For chaining: alpha can exceed 1.0 (chains grow longer).
- For open addressing: alpha must stay below 1.0 (typically under 0.75).
- When alpha exceeds a threshold, the table is **rehashed** to a larger capacity.

### Rehashing

When the load factor exceeds the threshold, the table:
1. Allocates a new array of roughly double the size (often the next prime).
2. Re-inserts every existing entry into the new array using the hash function.
3. Frees the old array.

Rehashing costs O(n) but happens infrequently, giving O(1) amortized insertion.

## Visual Representation

### Hash Table with Chaining

```
Hash Function: hash(key) = key MOD 7

Bucket Array:
Index  Chain
  0    --> [21, "val"] --> [7, "val"] --> NULL
  1    --> [15, "val"] --> NULL
  2    --> NULL
  3    --> [10, "val"] --> [3, "val"] --> [24, "val"] --> NULL
  4    --> [11, "val"] --> NULL
  5    --> [5, "val"] --> NULL
  6    --> NULL

  21 MOD 7 = 0,  7 MOD 7 = 0  (collision in bucket 0)
  10 MOD 7 = 3,  3 MOD 7 = 3,  24 MOD 7 = 3  (3-way collision in bucket 3)
```

### Hash Table with Open Addressing (Linear Probing)

```
Hash Function: hash(key) = key MOD 8

Insert order: 42, 18, 10, 34

  42 MOD 8 = 2  -->  slot 2 (empty, insert)
  18 MOD 8 = 2  -->  slot 2 (occupied), try 3 (empty, insert)
  10 MOD 8 = 2  -->  slot 2 (occupied), try 3 (occupied), try 4 (empty, insert)
  34 MOD 8 = 2  -->  slot 2, 3, 4 (all occupied), try 5 (empty, insert)

Index:   0     1     2     3     4     5     6     7
       +-----+-----+-----+-----+-----+-----+-----+-----+
       |     |     | 42  | 18  | 10  | 34  |     |     |
       +-----+-----+-----+-----+-----+-----+-----+-----+
                     ^     ^     ^     ^
                     |     |     |     +-- 34 probed here (4th try)
                     |     |     +-- 10 probed here (3rd try)
                     |     +-- 18 probed here (2nd try)
                     +-- 42 hashed directly here

Note: This clustering is called "primary clustering" -- a known
weakness of linear probing.
```

### Quadratic Probing

```
Probe sequence for hash(key) = h:
  Attempt 0: h
  Attempt 1: (h + 1^2) MOD capacity = (h + 1) MOD capacity
  Attempt 2: (h + 2^2) MOD capacity = (h + 4) MOD capacity
  Attempt 3: (h + 3^2) MOD capacity = (h + 9) MOD capacity

This spreads probes more widely than linear probing, reducing clustering.
```

### Rehashing

```
BEFORE (capacity=4, load factor=0.75, threshold exceeded):

Index:  0     1     2     3
      +-----+-----+-----+-----+
      | "a" | "b" | "c" |     |
      +-----+-----+-----+-----+
      entries=3, alpha=3/4=0.75

AFTER rehashing (capacity=8):

Index:  0     1     2     3     4     5     6     7
      +-----+-----+-----+-----+-----+-----+-----+-----+
      |     | "a" |     |     |     | "b" | "c" |     |
      +-----+-----+-----+-----+-----+-----+-----+-----+
      entries=3, alpha=3/8=0.375

All entries re-hashed with new capacity.
```

## Operations and Pseudocode

### Hash Table with Chaining

```
STRUCTURE Entry:
    key: KeyType
    value: ValueType
    next: Entry           // for chaining
END STRUCTURE

STRUCTURE HashTable:
    buckets: Array of Entry
    capacity: Integer
    count: Integer
    loadThreshold: Float   // e.g., 0.75
END STRUCTURE

FUNCTION createHashTable(capacity: Integer) -> HashTable:
    SET table = NEW HashTable
    SET table.buckets = ALLOCATE Array of size capacity (all NULL)
    SET table.capacity = capacity
    SET table.count = 0
    SET table.loadThreshold = 0.75
    RETURN table
END FUNCTION
```

### Insert (Chaining)

```
FUNCTION insert(table: HashTable, key: KeyType, value: ValueType) -> Void:
    IF (table.count + 1) / table.capacity > table.loadThreshold THEN
        CALL rehash(table)
    END IF

    SET index = hash(key) MOD table.capacity
    SET current = table.buckets[index]

    // Check if key already exists (update)
    WHILE current != NULL DO
        IF current.key == key THEN
            SET current.value = value
            RETURN
        END IF
        SET current = current.next
    END WHILE

    // Insert new entry at head of chain
    SET newEntry = NEW Entry(key, value)
    SET newEntry.next = table.buckets[index]
    SET table.buckets[index] = newEntry
    SET table.count = table.count + 1
END FUNCTION
```

### Search (Chaining)

```
FUNCTION search(table: HashTable, key: KeyType) -> ValueType:
    SET index = hash(key) MOD table.capacity
    SET current = table.buckets[index]

    WHILE current != NULL DO
        IF current.key == key THEN
            RETURN current.value
        END IF
        SET current = current.next
    END WHILE

    RETURN NOT_FOUND
END FUNCTION
```

### Delete (Chaining)

```
FUNCTION delete(table: HashTable, key: KeyType) -> Boolean:
    SET index = hash(key) MOD table.capacity
    SET current = table.buckets[index]
    SET previous = NULL

    WHILE current != NULL DO
        IF current.key == key THEN
            IF previous == NULL THEN
                SET table.buckets[index] = current.next
            ELSE
                SET previous.next = current.next
            END IF
            FREE current
            SET table.count = table.count - 1
            RETURN TRUE
        END IF
        SET previous = current
        SET current = current.next
    END WHILE

    RETURN FALSE
END FUNCTION
```

### Rehash

```
FUNCTION rehash(table: HashTable) -> Void:
    SET oldBuckets = table.buckets
    SET oldCapacity = table.capacity
    SET table.capacity = oldCapacity * 2
    SET table.buckets = ALLOCATE Array of size table.capacity (all NULL)
    SET table.count = 0

    FOR i FROM 0 TO oldCapacity - 1 DO
        SET current = oldBuckets[i]
        WHILE current != NULL DO
            CALL insert(table, current.key, current.value)
            SET current = current.next
        END WHILE
    END FOR

    FREE oldBuckets
END FUNCTION
```

### Insert with Linear Probing (Open Addressing)

```
FUNCTION insertLinearProbe(table: HashTable, key: KeyType, value: ValueType) -> Void:
    IF table.count / table.capacity > table.loadThreshold THEN
        CALL rehash(table)
    END IF

    SET index = hash(key) MOD table.capacity
    SET i = 0

    WHILE i < table.capacity DO
        SET probeIndex = (index + i) MOD table.capacity
        IF table.slots[probeIndex] is EMPTY or DELETED THEN
            SET table.slots[probeIndex] = NEW Entry(key, value)
            SET table.count = table.count + 1
            RETURN
        ELSE IF table.slots[probeIndex].key == key THEN
            SET table.slots[probeIndex].value = value
            RETURN
        END IF
        SET i = i + 1
    END WHILE

    ERROR "Hash table is full"
END FUNCTION
```

### Search with Linear Probing

```
FUNCTION searchLinearProbe(table: HashTable, key: KeyType) -> ValueType:
    SET index = hash(key) MOD table.capacity
    SET i = 0

    WHILE i < table.capacity DO
        SET probeIndex = (index + i) MOD table.capacity
        IF table.slots[probeIndex] is EMPTY THEN
            RETURN NOT_FOUND
        ELSE IF table.slots[probeIndex] is not DELETED
            AND table.slots[probeIndex].key == key THEN
            RETURN table.slots[probeIndex].value
        END IF
        SET i = i + 1
    END WHILE

    RETURN NOT_FOUND
END FUNCTION
```

### Delete with Tombstone (Open Addressing)

```
FUNCTION deleteLinearProbe(table: HashTable, key: KeyType) -> Boolean:
    SET index = hash(key) MOD table.capacity
    SET i = 0

    WHILE i < table.capacity DO
        SET probeIndex = (index + i) MOD table.capacity
        IF table.slots[probeIndex] is EMPTY THEN
            RETURN FALSE
        ELSE IF table.slots[probeIndex] is not DELETED
            AND table.slots[probeIndex].key == key THEN
            MARK table.slots[probeIndex] as DELETED    // tombstone
            SET table.count = table.count - 1
            RETURN TRUE
        END IF
        SET i = i + 1
    END WHILE

    RETURN FALSE
END FUNCTION
```

## Complexity Analysis

| Operation     | Best Case | Average Case | Worst Case | Notes                     |
|---------------|-----------|--------------|------------|---------------------------|
| Insert        | O(1)      | O(1)         | O(n)       | Worst: all keys collide   |
| Search        | O(1)      | O(1)         | O(n)       | Worst: long chain/cluster |
| Delete        | O(1)      | O(1)         | O(n)       | Worst: long chain/cluster |
| Rehash        | O(n)      | O(n)         | O(n)       | Redistributes all entries |
| Space         | -         | O(n)         | O(n)       | Plus unused capacity      |

| Factor                    | Chaining            | Open Addressing       |
|---------------------------|---------------------|-----------------------|
| Load factor range         | Can exceed 1.0      | Must be < 1.0         |
| Memory overhead           | Pointer per entry   | None (inline storage) |
| Cache performance         | Poor (linked lists) | Good (array)          |
| Clustering                | None                | Primary/secondary     |
| Deletion complexity       | Simple              | Requires tombstones   |
| Performance at high load  | Degrades gradually  | Degrades sharply      |

## When to Use / When NOT to Use

### Use Hash Tables When:
- You need fast average-case lookups, insertions, and deletions.
- Keys have a good hash function available.
- Ordering of elements is not required.
- You are building caches, indexes, or symbol tables.

### Avoid Hash Tables When:
- You need ordered traversal (use a balanced BST or sorted array).
- Keys are difficult to hash effectively.
- Worst-case O(n) performance is unacceptable (use a balanced BST for O(log n)).
- Memory is extremely constrained (hash tables waste capacity for low load factors).
- You need range queries (keys between A and B).

## Real-World Applications

- **Dictionaries/Maps**: Key-value stores in virtually every program.
- **Database indexing**: Hash indexes for equality lookups.
- **Caches**: Memoization, LRU caches, HTTP caches.
- **Symbol tables**: Compilers store variable names and types.
- **Spell checkers**: Dictionary of valid words for O(1) lookup.
- **Counting frequencies**: Count occurrences of words, characters, or events.
- **Deduplication**: Detect and remove duplicates in data streams.
- **Routers**: IP routing tables for fast destination lookup.

## Common Pitfalls and Best Practices

1. **Poor hash functions**: A hash function that clusters keys into a few buckets
   degrades performance to O(n). Always use well-tested hash algorithms.
2. **Ignoring load factor**: Letting the load factor grow too high causes excessive
   collisions. Rehash when alpha exceeds 0.7 - 0.75.
3. **Mutable keys**: If a key is modified after insertion, its hash may change,
   making it impossible to find. Always use immutable keys.
4. **Tombstone accumulation**: In open addressing, too many tombstones degrade
   search performance. Periodically rehash to clean them up.
5. **Not sizing to a prime**: For division-method hashing, a prime capacity reduces
   collision patterns. Avoid powers of two unless using multiplicative hashing.
6. **Hash flooding attacks**: An adversary can craft keys that all hash to the same
   bucket, causing O(n) lookups. Use randomized or cryptographic hash functions for
   untrusted input.
7. **Forgetting rehash cost**: While amortized O(1), a single rehash is O(n). In
   latency-sensitive systems, consider incremental rehashing.

## Comparison with Related Structures

| Feature              | Hash Table   | Balanced BST | Sorted Array | Trie          |
|----------------------|--------------|--------------|--------------|---------------|
| Search               | O(1) avg     | O(log n)     | O(log n)     | O(m)*         |
| Insert               | O(1) avg     | O(log n)     | O(n)         | O(m)*         |
| Delete               | O(1) avg     | O(log n)     | O(n)         | O(m)*         |
| Ordered traversal    | No           | Yes          | Yes          | Yes           |
| Range queries        | No           | Yes          | Yes          | Yes (prefix)  |
| Worst-case search    | O(n)         | O(log n)     | O(log n)     | O(m)*         |
| Space overhead       | High         | Moderate     | Low          | High          |
| Cache friendliness   | Moderate     | Poor         | Excellent    | Poor          |

\* m = length of the key (for string keys).

## Practice Problems

1. **Two Sum**: Given an array and a target, find two numbers that add up to the
   target. Use a hash table for O(n) time.

2. **Group Anagrams**: Given a list of strings, group anagrams together. Use sorted
   strings as hash keys.

3. **First Non-Repeating Character**: Find the first character in a string that
   does not repeat. Use a hash table to count frequencies.

4. **LRU Cache**: Implement a cache with O(1) get and put using a hash table
   combined with a doubly linked list.

5. **Implement a Hash Table**: Build a hash table from scratch with chaining,
   supporting insert, search, delete, and automatic rehashing.

6. **Consistent Hashing**: Design a hash ring that distributes keys across servers
   and handles server additions/removals with minimal key redistribution.
