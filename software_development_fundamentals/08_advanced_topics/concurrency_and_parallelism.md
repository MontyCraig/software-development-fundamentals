# Concurrency and Parallelism

## Overview

Concurrency and parallelism are fundamental concepts for building responsive, high-performance
software systems. While often used interchangeably in casual conversation, they represent
distinct ideas. Concurrency is about dealing with multiple tasks at once -- structuring a
program so that several logical tasks can make progress, even if only one is physically
executing at any given instant. Parallelism is about doing multiple tasks at once --
physically executing computations simultaneously on multiple processing units.

Understanding the distinction matters because the challenges differ. Concurrent programs must
handle shared state, synchronization, and ordering. Parallel programs must handle work
distribution, load balancing, and data partitioning. Many real-world systems require both:
a web server concurrently handles thousands of connections, while a scientific simulation
runs computations in parallel across many cores.

This chapter covers the core abstractions (threads, processes, async tasks), the problems
that arise (race conditions, deadlocks), the tools to solve them (mutexes, semaphores,
monitors), and higher-level patterns (producer-consumer, actor model, thread pools) that
make concurrent and parallel programming tractable.

## Key Concepts

### Concurrency vs Parallelism

```
  CONCURRENCY (structure)              PARALLELISM (execution)
  ========================             ========================

  Task A:  ===  ===  ===               Task A:  ===============
  Task B:    ===  ===  ===             Task B:  ===============
  Task C:      ===  ===               Task C:  ===============
  --------------------------           --------------------------
  Time -->  (interleaved on            Time -->  (simultaneous on
             one core)                            multiple cores)
```

Concurrency is a property of the program design. Parallelism is a property of the execution
environment. A concurrent program may or may not run in parallel depending on available
hardware. A parallel program is inherently concurrent.

### Threads vs Processes

```
  PROCESS A                      PROCESS B
  +------------------------+     +------------------------+
  | Own Memory Space        |     | Own Memory Space        |
  |  +------+ +------+     |     |  +------+ +------+     |
  |  |Thread| |Thread|     |     |  |Thread| |Thread|     |
  |  |  1   | |  2   |     |     |  |  3   | |  4   |     |
  |  +------+ +------+     |     |  +------+ +------+     |
  |  Shared Heap            |     |  Shared Heap            |
  |  +------------------+   |     |  +------------------+   |
  |  | Variables, Objects|   |     |  | Variables, Objects|   |
  |  +------------------+   |     |  +------------------+   |
  +------------------------+     +------------------------+
         |                               |
         +--- IPC (pipes, sockets, shared memory) ---+
```

A **process** has its own isolated memory space. Communication between processes requires
explicit inter-process communication (IPC). A **thread** shares memory with other threads
in the same process, making communication easy but synchronization essential.

| Aspect          | Process                  | Thread                   |
|-----------------|--------------------------|--------------------------|
| Memory          | Isolated                 | Shared within process    |
| Creation cost   | High                     | Low                      |
| Communication   | IPC required             | Shared memory            |
| Failure impact  | Isolated                 | Can crash entire process |
| Synchronization | Not needed (usually)     | Critical                 |

### Race Conditions

A race condition occurs when the outcome of a program depends on the unpredictable timing
of concurrent operations accessing shared state.

```
SHARED: counter = 0

THREAD A:                    THREAD B:
  SET temp = counter           SET temp = counter
  // temp = 0                  // temp = 0
  SET temp = temp + 1          SET temp = temp + 1
  // temp = 1                  // temp = 1
  SET counter = temp           SET counter = temp
  // counter = 1               // counter = 1

EXPECTED: counter = 2
ACTUAL:   counter = 1  (lost update!)
```

```
FUNCTION unsafe_increment(shared counter):
    SET temp = counter
    SET temp = temp + 1
    SET counter = temp
END FUNCTION

FUNCTION safe_increment(shared counter, lock: Mutex):
    ACQUIRE lock
    SET temp = counter
    SET temp = temp + 1
    SET counter = temp
    RELEASE lock
END FUNCTION
```

### Synchronization Primitives

#### Mutex (Mutual Exclusion)

A mutex allows only one thread to enter a critical section at a time.

```
FUNCTION transfer(from: Account, to: Account, amount: Number):
    ACQUIRE from.lock
    ACQUIRE to.lock

    IF from.balance >= amount THEN
        SET from.balance = from.balance - amount
        SET to.balance = to.balance + amount
    END IF

    RELEASE to.lock
    RELEASE from.lock
END FUNCTION
```

#### Semaphore

A semaphore maintains a count, allowing up to N threads to access a resource concurrently.

```
FUNCTION semaphore_example():
    SET pool_semaphore = CREATE Semaphore(max_count: 5)

    // Each worker must acquire before using the pool
    FUNCTION worker():
        WAIT pool_semaphore          // Decrement count (blocks if 0)
        use_connection_from_pool()
        SIGNAL pool_semaphore        // Increment count
    END FUNCTION
END FUNCTION
```

#### Monitor

A monitor combines a mutex with condition variables, enabling threads to wait for specific
conditions within a synchronized block.

```
MONITOR BoundedBuffer:
    DATA: buffer[], capacity, count = 0
    CONDITIONS: not_full, not_empty

    FUNCTION put(item):
        WHILE count == capacity DO
            WAIT not_full
        END WHILE
        APPEND item TO buffer
        SET count = count + 1
        SIGNAL not_empty
    END FUNCTION

    FUNCTION get() -> Item:
        WHILE count == 0 DO
            WAIT not_empty
        END WHILE
        SET item = REMOVE_FIRST FROM buffer
        SET count = count - 1
        SIGNAL not_full
        RETURN item
    END FUNCTION
END MONITOR
```

### Deadlock

Deadlock occurs when two or more threads are each waiting for the other to release a resource.

```
  Thread A                 Thread B
  +----------+             +----------+
  | Holds    |   wants     | Holds    |
  | Lock 1   |------------>| Lock 2   |
  |          |<------------|          |
  +----------+   wants     +----------+

  Both threads blocked forever!
```

#### Four Necessary Conditions (Coffman Conditions)

1. **Mutual Exclusion** -- A resource can only be held by one thread at a time
2. **Hold and Wait** -- A thread holds resources while waiting for others
3. **No Preemption** -- Resources cannot be forcibly taken from a thread
4. **Circular Wait** -- A cycle exists in the resource-dependency graph

#### Deadlock Prevention

Break any one of the four conditions:

```
FUNCTION deadlock_free_transfer(from: Account, to: Account, amount: Number):
    // Prevention: impose total ordering on locks to break circular wait
    IF from.id < to.id THEN
        SET first_lock = from.lock
        SET second_lock = to.lock
    ELSE
        SET first_lock = to.lock
        SET second_lock = from.lock
    END IF

    ACQUIRE first_lock
    ACQUIRE second_lock

    IF from.balance >= amount THEN
        SET from.balance = from.balance - amount
        SET to.balance = to.balance + amount
    END IF

    RELEASE second_lock
    RELEASE first_lock
END FUNCTION
```

#### Deadlock Detection

```
FUNCTION detect_deadlock(wait_for_graph: Graph) -> Boolean:
    // If the wait-for graph contains a cycle, deadlock exists
    RETURN has_cycle(wait_for_graph)
END FUNCTION
```

### Producer-Consumer Pattern

```
  +----------+    +-----------------+    +----------+
  | Producer | -> | Bounded Buffer  | -> | Consumer |
  +----------+    | [  |  |  |  ]   |    +----------+
  | Producer | -> |  capacity = 4   | -> | Consumer |
  +----------+    +-----------------+    +----------+
```

```
SHARED: queue = CREATE BoundedQueue(capacity: 10)

FUNCTION producer():
    LOOP
        SET item = produce_item()
        ACQUIRE queue.lock
        WHILE queue.is_full() DO
            WAIT queue.not_full_condition
        END WHILE
        queue.enqueue(item)
        SIGNAL queue.not_empty_condition
        RELEASE queue.lock
    END LOOP
END FUNCTION

FUNCTION consumer():
    LOOP
        ACQUIRE queue.lock
        WHILE queue.is_empty() DO
            WAIT queue.not_empty_condition
        END WHILE
        SET item = queue.dequeue()
        SIGNAL queue.not_full_condition
        RELEASE queue.lock
        process_item(item)
    END LOOP
END FUNCTION
```

### Readers-Writers Problem

Multiple readers can read simultaneously, but writers need exclusive access.

```
SHARED: read_count = 0, resource_lock = Mutex, read_count_lock = Mutex

FUNCTION reader():
    ACQUIRE read_count_lock
    SET read_count = read_count + 1
    IF read_count == 1 THEN
        ACQUIRE resource_lock      // First reader locks out writers
    END IF
    RELEASE read_count_lock

    read_data()

    ACQUIRE read_count_lock
    SET read_count = read_count - 1
    IF read_count == 0 THEN
        RELEASE resource_lock      // Last reader lets writers in
    END IF
    RELEASE read_count_lock
END FUNCTION

FUNCTION writer():
    ACQUIRE resource_lock
    write_data()
    RELEASE resource_lock
END FUNCTION
```

### Async/Await Patterns

Async/await provides concurrency without explicit thread management, ideal for I/O-bound
work.

```
ASYNC FUNCTION fetch_user_dashboard(user_id: String) -> Dashboard:
    // Launch concurrent I/O operations
    SET profile_future = ASYNC fetch_profile(user_id)
    SET orders_future  = ASYNC fetch_orders(user_id)
    SET prefs_future   = ASYNC fetch_preferences(user_id)

    // Await all results (runs concurrently)
    SET profile = AWAIT profile_future
    SET orders  = AWAIT orders_future
    SET prefs   = AWAIT prefs_future

    RETURN CREATE Dashboard(profile, orders, prefs)
END FUNCTION
```

### Thread Pools

A thread pool reuses a fixed set of threads to execute tasks, avoiding the overhead of
creating and destroying threads.

```
  Task Queue                    Thread Pool
  +--------+--------+-----+    +--------+--------+--------+
  | Task 5 | Task 4 | ... | -> | Worker | Worker | Worker |
  +--------+--------+-----+    |   1    |   2    |   3    |
                                +--------+--------+--------+
                                    |        |        |
                                  (execute tasks from queue)
```

```
FUNCTION create_thread_pool(size: Integer) -> ThreadPool:
    SET pool = CREATE ThreadPool
    SET pool.task_queue = CREATE BlockingQueue
    SET pool.workers = EMPTY LIST

    FOR i FROM 1 TO size DO
        SET worker = CREATE Thread(FUNCTION():
            LOOP
                SET task = pool.task_queue.take()    // Blocks if empty
                TRY
                    task.execute()
                CATCH error
                    log_error(error)
                END TRY
            END LOOP
        END FUNCTION)
        APPEND worker TO pool.workers
        START worker
    END FOR

    RETURN pool
END FUNCTION

FUNCTION submit_task(pool: ThreadPool, task: Task):
    pool.task_queue.put(task)
END FUNCTION
```

### Actor Model

In the actor model, each actor is an independent unit with its own state that communicates
exclusively through asynchronous messages. No shared mutable state exists.

```
  +----------+   message    +----------+   message    +----------+
  | Actor A  | -----------> | Actor B  | -----------> | Actor C  |
  | [state]  |              | [state]  |              | [state]  |
  | [mailbox]| <----------- | [mailbox]| <----------- | [mailbox]|
  +----------+   reply      +----------+   reply      +----------+
```

```
ACTOR CounterActor:
    STATE: count = 0

    ON RECEIVE message:
        MATCH message.type:
            CASE "increment":
                SET count = count + 1
            CASE "get_count":
                SEND message.sender, count
            CASE "reset":
                SET count = 0
        END MATCH
    END RECEIVE
END ACTOR

// Usage: no locks needed -- each actor processes one message at a time
SEND counter_actor, { type: "increment" }
SEND counter_actor, { type: "increment" }
SEND counter_actor, { type: "get_count", sender: self }
```

## Common Pitfalls and Best Practices

| Pitfall                        | Best Practice                                     |
|--------------------------------|---------------------------------------------------|
| Forgetting to release locks    | Use try/finally or RAII-style scoped locks         |
| Locking too broadly            | Minimize critical section size                     |
| Busy-waiting (spin locks)      | Use condition variables or blocking queues          |
| Thread-per-request             | Use thread pools with bounded size                  |
| Shared mutable state           | Prefer immutable data or message passing            |
| Ignoring thread safety of libs | Check documentation for thread-safety guarantees    |
| Over-parallelizing             | Profile first; parallelism has overhead             |

## Real-World Applications

- **Web servers**: Concurrently handle thousands of simultaneous client connections
- **Database systems**: Parallel query execution and concurrent transaction processing
- **GUI applications**: Background threads keep the interface responsive
- **Scientific computing**: Parallel matrix operations and simulations
- **Streaming platforms**: Concurrent video encoding pipelines

## Related Topics

- [Memory Management](memory_management.md) -- Thread stacks and shared heap concerns
- [System Design Basics](system_design_basics.md) -- Distributed concurrency
- [Networking Fundamentals](networking_fundamentals.md) -- Async I/O for network operations

## Practice Problems

1. Implement a thread-safe counter using a mutex. Verify that 1000 concurrent increments
   produce exactly 1000.

2. Write a producer-consumer system with a bounded buffer of size 5. Use two producers and
   three consumers. Ensure no item is processed twice and none is lost.

3. Given two accounts, implement a transfer function that is deadlock-free regardless of
   the order in which accounts are passed.

4. Design a read-write lock that allows multiple concurrent readers but exclusive writers.
   Ensure writers do not starve.

5. Build a simple thread pool that accepts tasks and distributes them across a fixed number
   of workers. Include graceful shutdown.

6. Model a chat room using the actor pattern: each user is an actor, and the room broadcasts
   messages. No shared state should exist.

7. Identify and fix the race condition in the following pseudocode:
   ```
   SHARED: list = []
   FUNCTION add_if_absent(item):
       IF item NOT IN list THEN
           APPEND item TO list
       END IF
   END FUNCTION
   ```
