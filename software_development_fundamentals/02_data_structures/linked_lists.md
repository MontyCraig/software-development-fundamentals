# Linked Lists

## Overview

A linked list is a linear data structure in which elements (called **nodes**) are
not stored in contiguous memory. Instead, each node contains a data field and one or
more **pointers** (or references) to other nodes. This design allows efficient
insertions and deletions at any position without shifting elements, at the cost of
losing constant-time random access.

Linked lists come in several variants. A **singly linked list** has nodes that point
only to the next node. A **doubly linked list** adds a pointer to the previous node,
enabling efficient backward traversal and deletion without a reference to the
predecessor. A **circular linked list** connects the last node back to the first,
forming a ring that is useful for round-robin scheduling and buffer management.

Linked lists are foundational to many higher-level structures. Stacks and queues can
be implemented with linked lists. Hash table chaining uses linked lists for buckets.
Adjacency lists for graphs are often linked lists. Understanding their pointer
mechanics is essential for working with trees, graphs, and memory management.

## Key Concepts

### Node Structure

Each node in a singly linked list holds:
- **data**: The stored value.
- **next**: A pointer to the following node, or NULL if this is the last node.

A doubly linked list node adds:
- **prev**: A pointer to the preceding node, or NULL if this is the first node.

### Head and Tail Pointers

The list maintains a **head** pointer to the first node. Many implementations also
maintain a **tail** pointer to the last node, enabling O(1) append operations.

### Sentinel (Dummy) Nodes

A **sentinel node** is a dummy node that does not hold meaningful data but simplifies
boundary conditions. With a sentinel head, you never have to special-case an empty
list or insertion at the front, because the sentinel is always present.

```
Sentinel-based doubly linked list (empty):
    +----------+     +----------+
    | SENTINEL |<--->| SENTINEL |
    |  (head)  |     |  (tail)  |
    +----------+     +----------+

After inserting A, B:
    +----------+     +---+     +---+     +----------+
    | SENTINEL |<--->| A |<--->| B |<--->| SENTINEL |
    |  (head)  |     +---+     +---+     |  (tail)  |
    +----------+                         +----------+
```

### Memory Characteristics

Unlike arrays, linked list nodes can be scattered anywhere in memory. This means:
- No wasted capacity (each node is allocated individually).
- Poor cache locality (nodes are not adjacent in memory).
- Extra memory overhead for pointer storage in every node.

## Visual Representation

### Singly Linked List

```
head
  |
  v
+------+---+     +------+---+     +------+---+     +------+------+
| data | o-+---->| data | o-+---->| data | o-+---->| data | NULL |
|  42  |   |     |  17  |   |     |  93  |   |     |   8  |      |
+------+---+     +------+---+     +------+---+     +------+------+

Each node: [data | next pointer]
Last node's next = NULL (end of list)
```

### Doubly Linked List

```
 NULL                                                      NULL
  ^                                                         ^
  |                                                         |
+----+------+---+     +---+------+---+     +---+------+----+
|prev| data |nxt+---->|prv| data |nxt+---->|prv| data |next|
|    |  42  |   |<----+   |  17  |   |<----+   |  93  |    |
+----+------+---+     +---+------+---+     +---+------+----+
  ^                                                  ^
  |                                                  |
 head                                               tail
```

### Circular Singly Linked List

```
head
  |
  v
+------+---+     +------+---+     +------+---+
| data | o-+---->| data | o-+---->| data | o-+---+
|  42  |   |     |  17  |   |     |  93  |   |   |
+------+---+     +------+---+     +------+---+   |
  ^                                               |
  +-----------------------------------------------+
  (last node points back to head)
```

### Insertion at a Given Position (Singly)

```
BEFORE: Insert X after node B

  A ----> B ----> C ----> D
                 ^
Step 1: Create node X, set X.next = B.next (C)
Step 2: Set B.next = X

AFTER:
  A ----> B ----> X ----> C ----> D
```

### Deletion of a Node (Doubly)

```
BEFORE: Delete node B

  A <---> B <---> C

Step 1: Set A.next = C       (skip B going forward)
Step 2: Set C.prev = A       (skip B going backward)
Step 3: Free node B

AFTER:
  A <---> C
```

## Operations and Pseudocode

### Create a New Node

```
FUNCTION createNode(value: Element) -> Node:
    SET node = ALLOCATE Node
    SET node.data = value
    SET node.next = NULL
    SET node.prev = NULL          // doubly linked only
    RETURN node
END FUNCTION
```

### Insert at Head (Singly)

```
FUNCTION insertAtHead(list: LinkedList, value: Element) -> Void:
    SET newNode = CALL createNode(value)
    SET newNode.next = list.head
    SET list.head = newNode
    IF list.tail == NULL THEN
        SET list.tail = newNode
    END IF
    SET list.size = list.size + 1
END FUNCTION
```

### Insert at Tail (Singly)

```
FUNCTION insertAtTail(list: LinkedList, value: Element) -> Void:
    SET newNode = CALL createNode(value)
    IF list.tail != NULL THEN
        SET list.tail.next = newNode
    ELSE
        SET list.head = newNode
    END IF
    SET list.tail = newNode
    SET list.size = list.size + 1
END FUNCTION
```

### Insert After a Given Node (Doubly)

```
FUNCTION insertAfter(list: DoublyLinkedList, target: Node, value: Element) -> Void:
    SET newNode = CALL createNode(value)
    SET newNode.prev = target
    SET newNode.next = target.next
    IF target.next != NULL THEN
        SET target.next.prev = newNode
    ELSE
        SET list.tail = newNode
    END IF
    SET target.next = newNode
    SET list.size = list.size + 1
END FUNCTION
```

### Delete a Node (Doubly)

```
FUNCTION deleteNode(list: DoublyLinkedList, target: Node) -> Element:
    SET value = target.data
    IF target.prev != NULL THEN
        SET target.prev.next = target.next
    ELSE
        SET list.head = target.next
    END IF
    IF target.next != NULL THEN
        SET target.next.prev = target.prev
    ELSE
        SET list.tail = target.prev
    END IF
    FREE target
    SET list.size = list.size - 1
    RETURN value
END FUNCTION
```

### Search

```
FUNCTION search(list: LinkedList, target: Element) -> Node:
    SET current = list.head
    WHILE current != NULL DO
        IF current.data == target THEN
            RETURN current
        END IF
        SET current = current.next
    END WHILE
    RETURN NULL
END FUNCTION
```

### Traverse and Print

```
FUNCTION traverse(list: LinkedList) -> Void:
    SET current = list.head
    WHILE current != NULL DO
        PRINT current.data
        SET current = current.next
    END WHILE
END FUNCTION
```

### Reverse a Singly Linked List (Iterative)

```
FUNCTION reverse(list: LinkedList) -> Void:
    SET prev = NULL
    SET current = list.head
    SET list.tail = list.head
    WHILE current != NULL DO
        SET nextNode = current.next
        SET current.next = prev
        SET prev = current
        SET current = nextNode
    END WHILE
    SET list.head = prev
END FUNCTION
```

### Detect Cycle (Floyd's Tortoise and Hare)

```
FUNCTION hasCycle(list: LinkedList) -> Boolean:
    SET slow = list.head
    SET fast = list.head
    WHILE fast != NULL AND fast.next != NULL DO
        SET slow = slow.next
        SET fast = fast.next.next
        IF slow == fast THEN
            RETURN TRUE
        END IF
    END WHILE
    RETURN FALSE
END FUNCTION
```

## Complexity Analysis

| Operation              | Singly Linked | Doubly Linked | Notes                    |
|------------------------|---------------|---------------|--------------------------|
| Insert at head         | O(1)          | O(1)          |                          |
| Insert at tail         | O(1)*         | O(1)*         | *With tail pointer       |
| Insert after node      | O(1)          | O(1)          | Given a reference        |
| Delete head            | O(1)          | O(1)          |                          |
| Delete tail            | O(n)          | O(1)          | Singly needs predecessor |
| Delete given node      | O(n)          | O(1)          | Singly needs predecessor |
| Search                 | O(n)          | O(n)          |                          |
| Access by index        | O(n)          | O(n)          |                          |
| Reverse                | O(n)          | O(n)          |                          |
| Space per node         | O(1)          | O(1)          | +1 ptr singly, +2 doubly |
| Total space            | O(n)          | O(n)          |                          |

## When to Use / When NOT to Use

### Use Linked Lists When:
- You need frequent insertions and deletions at arbitrary positions.
- The collection size is highly dynamic and unpredictable.
- You do not need random access by index.
- You are implementing other structures (stacks, queues, hash chains).
- Memory fragmentation is acceptable and nodes vary in creation time.

### Avoid Linked Lists When:
- You need fast random access (use an array).
- Cache performance is critical (arrays have superior locality).
- Memory overhead per element is a concern (each node stores extra pointers).
- The collection is small enough that shifting array elements is negligible.

## Real-World Applications

- **Undo/Redo systems**: A doubly linked list of states allows forward and backward
  navigation.
- **Music playlists**: Circular doubly linked lists for next/previous track.
- **Memory allocators**: Free lists track available memory blocks.
- **Hash table chaining**: Each bucket is a linked list of colliding entries.
- **Polynomial arithmetic**: Terms stored as linked list nodes, sorted by exponent.
- **LRU cache**: A doubly linked list combined with a hash table for O(1) eviction.
- **Operating system task schedulers**: Circular lists for round-robin scheduling.

## Common Pitfalls and Best Practices

1. **Losing references**: Always save `next` before modifying pointers during
   reversal or deletion, or you will lose the rest of the list.
2. **Memory leaks**: Every allocated node must eventually be freed. Walking the list
   and freeing each node on destruction is essential.
3. **NULL dereference**: Always check for NULL before accessing `node.next` or
   `node.prev`.
4. **Forgetting edge cases**: Empty list, single-element list, and operations on
   head/tail all need special handling (or use sentinel nodes to eliminate them).
5. **Not using sentinel nodes**: Sentinels simplify code significantly for doubly
   linked lists by removing all NULL checks for head and tail.
6. **Cycle creation**: In singly linked lists, accidentally pointing a node's next
   to an earlier node creates an infinite loop. Use Floyd's algorithm to detect.

## Comparison with Related Structures

| Feature              | Singly Linked | Doubly Linked | Dynamic Array | Skip List  |
|----------------------|---------------|---------------|---------------|------------|
| Insert at head       | O(1)          | O(1)          | O(n)          | O(log n)   |
| Insert at tail       | O(1)*         | O(1)          | O(1) amort.   | O(log n)   |
| Delete at head       | O(1)          | O(1)          | O(n)          | O(log n)   |
| Delete at tail       | O(n)          | O(1)          | O(1)          | O(log n)   |
| Random access        | O(n)          | O(n)          | O(1)          | O(log n)   |
| Search               | O(n)          | O(n)          | O(n)          | O(log n)   |
| Memory per element   | +1 pointer    | +2 pointers   | None          | ~+1.33 ptr |
| Cache friendliness   | Poor          | Poor          | Excellent     | Poor       |

\* With tail pointer maintained.

## Practice Problems

1. **Find Middle Element**: Given a singly linked list, find the middle element in
   one pass using the slow/fast pointer technique.

2. **Merge Two Sorted Lists**: Given two sorted singly linked lists, merge them into
   one sorted list in O(n + m) time and O(1) extra space.

3. **Remove Nth Node from End**: Remove the nth node from the end of a singly linked
   list in one pass using two pointers spaced n nodes apart.

4. **Palindrome Check**: Determine if a singly linked list is a palindrome using
   O(n) time and O(1) space (reverse the second half, compare, restore).

5. **Flatten a Multilevel Doubly Linked List**: Given a doubly linked list where
   some nodes have a child pointer to another doubly linked list, flatten all levels
   into a single-level list.

6. **LRU Cache**: Implement a Least Recently Used cache using a doubly linked list
   and a hash table, supporting O(1) get and put operations.
