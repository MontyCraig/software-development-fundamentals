# Memory Management

## Overview

Memory management is one of the most critical aspects of software development. Every program
must allocate memory to store data, use that memory during computation, and eventually release
it when no longer needed. Poor memory management leads to leaks, crashes, corruption, and
security vulnerabilities. Understanding how memory works -- from the hardware level up through
language runtime abstractions -- is essential for writing reliable, efficient software.

At the lowest level, a program's memory is divided into distinct regions with different
lifetimes and access patterns. The stack provides fast, automatically managed memory for
local variables and function calls. The heap provides flexible, dynamically allocated memory
for objects whose size or lifetime is not known at compile time. Between manual management
and fully automatic garbage collection lies a spectrum of strategies, each with trade-offs
in performance, safety, and complexity.

This chapter covers memory layout, allocation strategies, garbage collection algorithms,
common memory errors, and optimization techniques including cache locality and memory pools.

## Key Concepts

### Stack vs Heap

```
  HIGH ADDRESS
  +---------------------------+
  |         STACK             |  <- Grows downward
  |  +---------------------+  |
  |  | main() frame        |  |
  |  |   local_var_a       |  |
  |  |   local_var_b       |  |
  |  +---------------------+  |
  |  | process() frame     |  |
  |  |   param_x           |  |
  |  |   local_result      |  |
  |  +---------------------+  |
  |         |  |              |
  |         v  ^              |
  |                           |
  |         ^  v              |
  |         |  |              |
  |  +---------------------+  |
  |  | Object A (64 bytes) |  |
  |  +---------------------+  |
  |  | Object B (128 bytes)|  |
  |  +---------------------+  |
  |  | [free]   [Object C] |  |
  |  +---------------------+  |
  |         HEAP              |  <- Grows upward
  +---------------------------+
  |     STATIC / GLOBAL       |
  +---------------------------+
  |     CODE (TEXT)            |
  +---------------------------+
  LOW ADDRESS
```

| Aspect           | Stack                         | Heap                            |
|------------------|-------------------------------|---------------------------------|
| Allocation       | Automatic (push/pop)          | Manual or GC-managed            |
| Speed            | Very fast (pointer adjust)    | Slower (search for free block)  |
| Size             | Limited (typically 1-8 MB)    | Large (limited by system RAM)   |
| Lifetime         | Function scope                | Until explicitly freed or GC'd  |
| Fragmentation    | None                          | Can fragment over time          |
| Thread safety    | Each thread has own stack     | Shared; needs synchronization   |

### Allocation and Deallocation

```
FUNCTION demonstrate_memory():
    // Stack allocation -- automatic
    SET x = 42                          // Allocated on stack
    SET name = "hello"                  // Allocated on stack (or constant pool)

    // Heap allocation -- explicit
    SET buffer = ALLOCATE(1024 bytes)   // Allocated on heap

    process(buffer)

    DEALLOCATE(buffer)                  // Must be freed explicitly
    // x and name are freed automatically when function returns
END FUNCTION
```

Common heap allocation strategies:

- **First Fit**: Scan free list, use first block large enough
- **Best Fit**: Find smallest block that fits (less waste, slower search)
- **Buddy System**: Split and merge power-of-two blocks

```
  Free List (First Fit):
  +------+    +----------+    +----+    +--------+
  | 32B  | -> | 128B     | -> | 8B | -> | 256B   |
  +------+    +----------+    +----+    +--------+

  Request: 100 bytes --> allocates from 128B block
```

### Garbage Collection Algorithms

#### Reference Counting

Each object maintains a count of how many references point to it. When the count reaches
zero, the object is freed.

```
FUNCTION reference_counting_example():
    SET a = CREATE Object()       // Object refcount = 1
    SET b = a                     // Object refcount = 2
    SET a = NULL                  // Object refcount = 1
    SET b = NULL                  // Object refcount = 0 --> freed!
END FUNCTION
```

```
  Object A (refcount: 2)         Object A (refcount: 0)
  ^         ^                    --> FREED
  |         |
  ref_1    ref_2                 ref_1 = NULL, ref_2 = NULL
```

**Problem**: Circular references are never freed.

```
  +----------+       +----------+
  | Object A | ----> | Object B |
  | refcnt=1 | <---- | refcnt=1 |
  +----------+       +----------+
  (No external references, but refcount never reaches 0!)
```

#### Mark-and-Sweep

Periodically traces all reachable objects from root references, then frees everything else.

```
FUNCTION mark_and_sweep(roots: List):
    // Phase 1: Mark
    SET worklist = COPY OF roots
    WHILE worklist IS NOT EMPTY DO
        SET obj = POP FROM worklist
        IF obj.marked == false THEN
            SET obj.marked = true
            FOR EACH ref IN obj.references DO
                PUSH ref ONTO worklist
            END FOR
        END IF
    END WHILE

    // Phase 2: Sweep
    FOR EACH obj IN all_heap_objects DO
        IF obj.marked == false THEN
            DEALLOCATE(obj)
        ELSE
            SET obj.marked = false     // Reset for next cycle
        END IF
    END FOR
END FUNCTION
```

```
  ROOT SET          HEAP
  +------+     +---+   +---+   +---+
  | root |---->| A |-->| B |   | D | (unreachable)
  +------+     +---+   +---+   +---+
                  |       |
                  v       v
                +---+   +---+
                | C |   | E | (reachable via B)
                +---+   +---+

  After sweep: A, B, C, E survive. D is freed.
```

#### Generational Garbage Collection

Based on the observation that most objects die young. Memory is divided into generations.

```
  +------------------+------------------+------------------+
  |   Young Gen      |   Old Gen        |   Permanent      |
  |  (collected      |  (collected      |  (rarely         |
  |   frequently)    |   infrequently)  |   collected)     |
  +------------------+------------------+------------------+
  | New objects here  | Survivors move   | Long-lived data  |
  | Minor GC: fast   | Major GC: slow   |                  |
  +------------------+------------------+------------------+

  Object lifecycle:
  ALLOCATE --> Young Gen --[survives N collections]--> Old Gen
```

```
FUNCTION generational_gc():
    // Minor collection: only scan young generation
    mark_and_sweep(young_generation_roots)

    // Promote survivors
    FOR EACH obj IN young_generation DO
        IF obj.survival_count > THRESHOLD THEN
            MOVE obj TO old_generation
        END IF
    END FOR

    // Major collection: scan everything (infrequent)
    IF old_generation.usage > HIGH_WATERMARK THEN
        mark_and_sweep(all_roots)
    END IF
END FUNCTION
```

### Memory Leaks

A memory leak occurs when allocated memory is no longer reachable by the program but has
not been freed. Over time, leaks consume all available memory.

Common causes:
- Forgetting to deallocate heap memory
- Holding references in long-lived collections (event listeners, caches)
- Circular references in reference-counted systems
- Unclosed resources (file handles, network connections)

```
FUNCTION leaky_function():
    LOOP 1000 TIMES
        SET data = ALLOCATE(1024 bytes)
        process(data)
        // BUG: forgot to DEALLOCATE(data)
    END LOOP
    // 1000 * 1024 = 1MB leaked per call!
END FUNCTION

FUNCTION fixed_function():
    LOOP 1000 TIMES
        SET data = ALLOCATE(1024 bytes)
        TRY
            process(data)
        FINALLY
            DEALLOCATE(data)
        END TRY
    END LOOP
END FUNCTION
```

### Dangling Pointers and References

A dangling pointer references memory that has been freed. Accessing it causes undefined
behavior -- crashes, corruption, or security vulnerabilities.

```
FUNCTION dangling_example():
    SET ptr = ALLOCATE(64 bytes)
    DEALLOCATE(ptr)
    // ptr is now dangling!
    WRITE TO ptr             // UNDEFINED BEHAVIOR
END FUNCTION
```

Prevention strategies:
- Set pointers to NULL after deallocation
- Use ownership models (one owner responsible for freeing)
- Use smart pointers / reference-counted handles
- Rely on garbage collection

### Memory Pools

A memory pool pre-allocates a large block and hands out fixed-size chunks, avoiding the
overhead of general-purpose allocation.

```
FUNCTION create_pool(chunk_size: Integer, count: Integer) -> Pool:
    SET pool.memory = ALLOCATE(chunk_size * count)
    SET pool.free_list = EMPTY LIST
    FOR i FROM 0 TO count - 1 DO
        APPEND (pool.memory + i * chunk_size) TO pool.free_list
    END FOR
    RETURN pool
END FUNCTION

FUNCTION pool_allocate(pool: Pool) -> Pointer:
    IF pool.free_list IS EMPTY THEN
        RAISE OutOfMemoryError
    END IF
    RETURN POP FROM pool.free_list
END FUNCTION

FUNCTION pool_deallocate(pool: Pool, ptr: Pointer):
    APPEND ptr TO pool.free_list
END FUNCTION
```

```
  Memory Pool (chunk_size = 32 bytes, count = 8):
  +----+----+----+----+----+----+----+----+
  | C1 | C2 | C3 | C4 | C5 | C6 | C7 | C8 |
  +----+----+----+----+----+----+----+----+
  [used][free][used][used][free][free][used][free]

  Free list: C2 -> C5 -> C6 -> C8
```

### Cache Locality

Modern CPUs use multi-level caches. Accessing memory in predictable, sequential patterns
(cache-friendly access) is dramatically faster than random access.

```
  CPU Core
  +------------------+
  | Registers (1 ns) |
  +------------------+
  | L1 Cache  (2 ns) |  <- 32-64 KB
  +------------------+
  | L2 Cache  (7 ns) |  <- 256 KB - 1 MB
  +------------------+
  | L3 Cache (20 ns) |  <- 4-32 MB
  +------------------+
  | Main RAM (100 ns) | <- 4-512 GB
  +------------------+
```

- **Temporal locality**: Recently accessed data is likely to be accessed again soon
- **Spatial locality**: Data near recently accessed data is likely to be accessed soon

```
// GOOD: Sequential access (spatial locality)
FUNCTION sum_array(arr: Array[N]) -> Number:
    SET total = 0
    FOR i FROM 0 TO N - 1 DO
        SET total = total + arr[i]     // Sequential memory access
    END FOR
    RETURN total
END FUNCTION

// BAD: Random access (poor cache behavior)
FUNCTION sum_linked_list(head: Node) -> Number:
    SET total = 0
    SET current = head
    WHILE current IS NOT NULL DO
        SET total = total + current.value   // Nodes scattered in heap
        SET current = current.next
    END WHILE
    RETURN total
END FUNCTION
```

### Virtual Memory

Virtual memory provides each process with the illusion of a large, contiguous address
space, even when physical memory is limited. The operating system and hardware map virtual
addresses to physical frames.

```
  Virtual Address Space (per process)    Physical Memory
  +------------------+                   +------------------+
  | Page 0           |----+              | Frame 0 (OS)     |
  +------------------+    |              +------------------+
  | Page 1           |----|--+           | Frame 1          |<--+
  +------------------+    |  |           +------------------+   |
  | Page 2           |    +--|---------->| Frame 2          |   |
  +------------------+       |           +------------------+   |
  | Page 3 (on disk) |      +---------->| Frame 3          |   |
  +------------------+                   +------------------+   |
  | Page 4           |--------------------------------------+
  +------------------+

  Page Table maps virtual pages -> physical frames (or disk)
```

When a page is not in physical memory, a **page fault** occurs, and the OS loads it from
disk -- orders of magnitude slower than a cache miss.

### Memory-Mapped I/O

Memory-mapped I/O maps a file or device directly into a process's virtual address space,
allowing file access through normal memory read/write operations.

```
FUNCTION process_large_file(path: String):
    SET mapped_region = MEMORY_MAP(path, mode: READ_ONLY)
    // File contents now accessible as a memory region
    FOR i FROM 0 TO mapped_region.size - 1 DO
        process_byte(mapped_region[i])    // No explicit read() calls
    END FOR
    UNMAP(mapped_region)
END FUNCTION
```

## Common Pitfalls and Best Practices

| Pitfall                          | Best Practice                                    |
|----------------------------------|--------------------------------------------------|
| Memory leaks                     | Always pair allocate with deallocate (or use GC)  |
| Dangling pointers                | Nullify after free; use ownership patterns         |
| Buffer overflow                  | Always check bounds before writing                 |
| Stack overflow (deep recursion)  | Use iteration or increase stack size               |
| Fragmentation                    | Use memory pools for fixed-size allocations         |
| Poor cache performance           | Use arrays over linked lists; access sequentially   |
| Excessive allocation             | Reuse objects; pool frequently allocated types      |

## Real-World Applications

- **Game engines**: Custom memory allocators and pools for frame-rate-sensitive allocation
- **Database systems**: Buffer pools map disk pages into memory for efficient access
- **Embedded systems**: No garbage collector; manual management with strict constraints
- **Web browsers**: Generational GC for short-lived DOM objects
- **Operating systems**: Virtual memory and page management

## Related Topics

- [Concurrency and Parallelism](concurrency_and_parallelism.md) -- Thread stacks and shared heap
- [Database Fundamentals](database_fundamentals.md) -- Buffer pool management
- [System Design Basics](system_design_basics.md) -- Caching strategies

## Practice Problems

1. Trace the execution of a mark-and-sweep collector on a heap containing six objects where
   only three are reachable from the root set.

2. Identify the memory leak in a function that allocates a buffer, processes it, and may
   return early on error without deallocating.

3. Design a fixed-size memory pool for objects of 48 bytes. Include allocation, deallocation,
   and a diagnostic function that reports utilization.

4. Explain why iterating over a two-dimensional array row-by-row is faster than column-by-
   column in a row-major memory layout.

5. Given a reference-counting system with two objects referencing each other, demonstrate
   why they leak and propose a solution (weak references).

6. Draw the virtual memory page table for a process with 8 virtual pages, where pages 0, 2,
   and 5 are in physical memory and the rest are on disk.
