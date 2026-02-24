# Queues

## Overview

A queue is a linear data structure that follows the **First-In, First-Out (FIFO)**
principle. The first element added to the queue is the first one to be removed, much
like people waiting in line -- whoever arrives first gets served first. Queues are
fundamental to scheduling, buffering, and breadth-first traversal algorithms.

The two primary operations are **enqueue** (add an element to the rear) and
**dequeue** (remove an element from the front). Both run in O(1) time in a properly
implemented queue. A **peek** (or front) operation returns the front element without
removing it.

Beyond the basic FIFO queue, several important variants exist. A **circular queue**
(ring buffer) uses a fixed-size array efficiently by wrapping around. A **deque**
(double-ended queue) allows insertion and removal at both ends. A **priority queue**
serves elements by priority rather than arrival order, typically backed by a heap.
Each variant serves distinct use cases while sharing the fundamental concept of
ordered processing.

## Key Concepts

### FIFO Ordering

FIFO means the element that has been waiting the longest is served next. This is the
natural model for any "fairness" scenario: task scheduling, print job management,
message passing, and network packet routing.

### Front and Rear Pointers

A queue maintains two pointers:
- **front**: Points to the next element to be dequeued.
- **rear**: Points to the position where the next element will be enqueued.

### Circular Queue (Ring Buffer)

A naive array-based queue wastes space as the front advances. A circular queue solves
this by wrapping the indices using modular arithmetic:

```
next_index = (current_index + 1) MOD capacity
```

This reuses vacated slots at the beginning of the array without shifting elements.

### Deque (Double-Ended Queue)

A deque supports four operations: insert at front, insert at rear, remove from front,
and remove from rear -- all in O(1) time. It generalizes both stacks and queues.

### Priority Queue

A priority queue dequeues the element with the highest (or lowest) priority rather
than the oldest. It is typically implemented with a binary heap rather than a linear
structure, giving O(log n) enqueue and dequeue.

## Visual Representation

### Basic Queue Operations

```
ENQUEUE A:   ENQUEUE B:   ENQUEUE C:   DEQUEUE:     DEQUEUE:
                                       (returns A)  (returns B)

front rear   front  rear  front  rear  front rear   front rear
  |    |       |     |      |     |      |    |       |    |
  v    v       v     v      v     v      v    v       v    v
+---+        +---+---+    +---+---+---+    +---+---+    +---+
| A |        | A | B |    | A | B | C |    | B | C |    | C |
+---+        +---+---+    +---+---+---+    +---+---+    +---+
```

### Circular Queue (Ring Buffer)

```
Capacity = 6, front = 2, rear = 5, count = 3

  Index:   0     1     2     3     4     5
         +-----+-----+-----+-----+-----+-----+
         |     |     | 42  | 17  | 93  |     |
         +-----+-----+-----+-----+-----+-----+
                       ^                 ^
                     front              rear

After ENQUEUE 11, ENQUEUE 7 (wraps around):

  Index:   0     1     2     3     4     5
         +-----+-----+-----+-----+-----+-----+
         |  7  |     | 42  | 17  | 93  | 11  |
         +-----+-----+-----+-----+-----+-----+
           ^     ^     ^
          rear  (gap) front

rear = (5 + 1) MOD 6 = 0, then (0 + 1) MOD 6 = 1
```

### Deque (Double-Ended Queue)

```
        front                              rear
          |                                  |
          v                                  v
        +------+------+------+------+------+
        |  A   |  B   |  C   |  D   |  E   |
        +------+------+------+------+------+
          ^                             ^
     removeFront                   removeRear
     insertFront                   insertRear
```

### Priority Queue (Conceptual)

```
INSERT items with priorities:

  (Task C, priority 1)  --> dequeue first  (lowest = highest priority)
  (Task A, priority 3)
  (Task B, priority 2)

Queue state (ordered by priority):
  FRONT [ Task C (1) | Task B (2) | Task A (3) ] REAR

DEQUEUE returns: Task C (priority 1)
```

### Linked-List Queue

```
  front                                        rear
    |                                            |
    v                                            v
+------+---+     +------+---+     +------+------+
|  42  | o-+---->|  17  | o-+---->|  93  | NULL |
+------+---+     +------+---+     +------+------+

Dequeue from front (head), Enqueue at rear (tail).
```

## Operations and Pseudocode

### Circular Queue (Array-Based)

```
STRUCTURE CircularQueue:
    data: Array
    front: Integer
    rear: Integer
    count: Integer
    capacity: Integer
END STRUCTURE

FUNCTION createQueue(capacity: Integer) -> CircularQueue:
    SET queue = NEW CircularQueue
    SET queue.data = ALLOCATE Array of size capacity
    SET queue.front = 0
    SET queue.rear = 0
    SET queue.count = 0
    SET queue.capacity = capacity
    RETURN queue
END FUNCTION

FUNCTION enqueue(queue: CircularQueue, value: Element) -> Void:
    IF queue.count == queue.capacity THEN
        ERROR "Queue overflow"
    END IF
    SET queue.data[queue.rear] = value
    SET queue.rear = (queue.rear + 1) MOD queue.capacity
    SET queue.count = queue.count + 1
END FUNCTION

FUNCTION dequeue(queue: CircularQueue) -> Element:
    IF queue.count == 0 THEN
        ERROR "Queue underflow"
    END IF
    SET value = queue.data[queue.front]
    SET queue.front = (queue.front + 1) MOD queue.capacity
    SET queue.count = queue.count - 1
    RETURN value
END FUNCTION

FUNCTION peek(queue: CircularQueue) -> Element:
    IF queue.count == 0 THEN
        ERROR "Queue is empty"
    END IF
    RETURN queue.data[queue.front]
END FUNCTION

FUNCTION isEmpty(queue: CircularQueue) -> Boolean:
    RETURN queue.count == 0
END FUNCTION

FUNCTION isFull(queue: CircularQueue) -> Boolean:
    RETURN queue.count == queue.capacity
END FUNCTION
```

### Linked-List Queue

```
FUNCTION enqueue(queue: LinkedQueue, value: Element) -> Void:
    SET newNode = CALL createNode(value)
    IF queue.rear != NULL THEN
        SET queue.rear.next = newNode
    ELSE
        SET queue.front = newNode
    END IF
    SET queue.rear = newNode
    SET queue.count = queue.count + 1
END FUNCTION

FUNCTION dequeue(queue: LinkedQueue) -> Element:
    IF queue.front == NULL THEN
        ERROR "Queue underflow"
    END IF
    SET value = queue.front.data
    SET oldFront = queue.front
    SET queue.front = queue.front.next
    IF queue.front == NULL THEN
        SET queue.rear = NULL
    END IF
    FREE oldFront
    SET queue.count = queue.count - 1
    RETURN value
END FUNCTION
```

### Deque (Array-Based)

```
FUNCTION insertFront(deque: Deque, value: Element) -> Void:
    IF deque.count == deque.capacity THEN
        CALL resize(deque)
    END IF
    SET deque.front = (deque.front - 1 + deque.capacity) MOD deque.capacity
    SET deque.data[deque.front] = value
    SET deque.count = deque.count + 1
END FUNCTION

FUNCTION insertRear(deque: Deque, value: Element) -> Void:
    IF deque.count == deque.capacity THEN
        CALL resize(deque)
    END IF
    SET deque.data[deque.rear] = value
    SET deque.rear = (deque.rear + 1) MOD deque.capacity
    SET deque.count = deque.count + 1
END FUNCTION

FUNCTION removeFront(deque: Deque) -> Element:
    IF deque.count == 0 THEN
        ERROR "Deque underflow"
    END IF
    SET value = deque.data[deque.front]
    SET deque.front = (deque.front + 1) MOD deque.capacity
    SET deque.count = deque.count - 1
    RETURN value
END FUNCTION

FUNCTION removeRear(deque: Deque) -> Element:
    IF deque.count == 0 THEN
        ERROR "Deque underflow"
    END IF
    SET deque.rear = (deque.rear - 1 + deque.capacity) MOD deque.capacity
    SET value = deque.data[deque.rear]
    SET deque.count = deque.count - 1
    RETURN value
END FUNCTION
```

### BFS Using a Queue

```
FUNCTION breadthFirstSearch(graph: Graph, start: Node) -> List:
    SET visited = EMPTY Set
    SET queue = EMPTY Queue
    SET result = EMPTY List

    CALL enqueue(queue, start)
    ADD start TO visited

    WHILE NOT isEmpty(queue) DO
        SET current = CALL dequeue(queue)
        APPEND current TO result

        FOR EACH neighbor IN graph.neighbors(current) DO
            IF neighbor NOT IN visited THEN
                ADD neighbor TO visited
                CALL enqueue(queue, neighbor)
            END IF
        END FOR
    END WHILE

    RETURN result
END FUNCTION
```

## Complexity Analysis

| Operation        | Circular Queue | Linked Queue | Deque (Array) | Priority Queue |
|------------------|----------------|--------------|---------------|----------------|
| Enqueue / Insert | O(1)           | O(1)         | O(1)*         | O(log n)       |
| Dequeue / Remove | O(1)           | O(1)         | O(1)*         | O(log n)       |
| Peek / Front     | O(1)           | O(1)         | O(1)          | O(1)           |
| isEmpty          | O(1)           | O(1)         | O(1)          | O(1)           |
| Search           | O(n)           | O(n)         | O(n)          | O(n)           |
| Space            | O(n)           | O(n)         | O(n)          | O(n)           |

\* Amortized O(1) when resizing is needed.

| Metric              | Array (Circular)     | Linked List          |
|----------------------|----------------------|----------------------|
| Memory per element   | Element only         | Element + 1 pointer  |
| Cache performance    | Excellent            | Poor                 |
| Fixed capacity       | Yes (unless resized) | No                   |
| Allocation overhead  | One block            | Per-node allocation  |

## When to Use / When NOT to Use

### Use Queues When:
- You need FIFO processing (first come, first served).
- Implementing BFS (breadth-first search) on graphs or trees.
- Buffering data between producers and consumers.
- Task scheduling (print queues, job schedulers, message queues).
- Level-order tree traversal.

### Use Deques When:
- You need both LIFO and FIFO access.
- Implementing sliding window algorithms.
- You need a work-stealing queue (take from one end, steal from the other).

### Use Priority Queues When:
- Elements must be processed by priority, not arrival order.
- Implementing Dijkstra's shortest path algorithm.
- Event-driven simulation with time-stamped events.
- Scheduling with priority levels.

### Avoid Queues When:
- You need random access to elements (use an array).
- You need LIFO access (use a stack).
- The ordering requirement is more complex than FIFO or priority.

## Real-World Applications

- **Print spoolers**: Documents are printed in the order they are submitted.
- **Web server request handling**: Requests are queued and processed in order.
- **Breadth-first search**: Graph traversal explores neighbors level by level.
- **Message queues**: Asynchronous communication between services (producer-consumer).
- **CPU scheduling**: Processes wait in a ready queue for CPU time.
- **Keyboard buffer**: Keystrokes are buffered in a queue until processed.
- **Streaming data**: Ring buffers for audio/video streaming.
- **Traffic management**: Vehicles queue at intersections and toll booths.

## Common Pitfalls and Best Practices

1. **Confusing full and empty**: In a circular queue without a count field, both
   full and empty states have `front == rear`. Always maintain a separate count or
   waste one slot to distinguish them.
2. **Forgetting modular arithmetic**: Without the MOD operation, circular queues
   will access out-of-bounds indices.
3. **Queue underflow**: Always check `isEmpty()` before dequeue or peek.
4. **Not handling the single-element case**: When dequeuing the last element from a
   linked-list queue, both `front` and `rear` must be set to NULL.
5. **Using a queue where a priority queue is needed**: If processing order depends
   on priority rather than arrival time, a plain queue will produce incorrect results.
6. **Memory leaks in linked queues**: Dequeued nodes must be freed explicitly in
   environments without automatic garbage collection.

## Comparison with Related Structures

| Feature              | Queue      | Stack      | Deque      | Priority Queue |
|----------------------|------------|------------|------------|----------------|
| Ordering             | FIFO       | LIFO       | Both       | By priority    |
| Insert location      | Rear       | Top        | Both ends  | Anywhere       |
| Remove location      | Front      | Top        | Both ends  | Min/Max        |
| Peek location        | Front      | Top        | Both ends  | Min/Max        |
| Insert complexity    | O(1)       | O(1)       | O(1)       | O(log n)       |
| Remove complexity    | O(1)       | O(1)       | O(1)       | O(log n)       |
| Primary use          | Scheduling | Parsing    | Windows    | Scheduling     |
| Common backing store | Array/List | Array/List | Array/List | Binary Heap    |

## Practice Problems

1. **Implement Queue Using Two Stacks**: Build a FIFO queue using only two LIFO
   stacks. Achieve amortized O(1) per operation.

2. **Circular Queue Implementation**: Implement a fixed-size circular queue with
   enqueue, dequeue, front, rear, isEmpty, and isFull operations.

3. **Sliding Window Maximum**: Given an array and window size k, find the maximum
   in each window as it slides from left to right. Use a deque for O(n) time.

4. **Number of Recent Calls**: Design a class that counts the number of requests
   received in the past 3000 milliseconds using a queue.

5. **Interleave Queue Halves**: Given a queue of even size, interleave its first
   half with its second half. Use only a stack as auxiliary storage.

6. **Design Hit Counter**: Design a system that counts hits received in the past
   5 minutes. Support `hit(timestamp)` and `getHits(timestamp)` operations using
   a circular queue approach.
