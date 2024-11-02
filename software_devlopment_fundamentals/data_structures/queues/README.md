# Queues

## Overview
A queue is a FIFO (First-In, First-Out) data structure that supports enqueue and dequeue operations.

## Types of Queues

### 1. Simple Queue
- Basic FIFO implementation
- Array-based and linked list-based versions
- Front and rear pointer management

### 2. Circular Queue
- Efficient memory utilization
- Handling wrap-around
- Full and empty state detection

### 3. Priority Queue
- Element priorities
- Heap implementation
- Applications in scheduling
- Time complexities for different operations

### 4. Double-ended Queue (Deque)
- Insert/delete at both ends
- Implementation considerations
- Applications

## Core Operations
- Enqueue: O(1)
- Dequeue: O(1)
- Front/Peek: O(1)
- isEmpty: O(1)
- isFull: O(1) (for array-based)

## Implementation Files
- `simple_queue.py`
- `circular_queue.py`
- `priority_queue.py`
- `deque.py`
- `queue_utils.py`

## Testing
- `test_simple_queue.py`
- `test_circular_queue.py`
- `test_priority_queue.py`
- `test_deque.py` 