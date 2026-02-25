# Section 8: Advanced Topics

## Overview

This section covers the cross-cutting concerns and systems-level knowledge that
distinguishes senior developers from juniors. These topics go beyond writing correct code
to address how software interacts with hardware, networks, databases, and other systems --
and how to design, secure, and scale those interactions.

Each topic is self-contained but interconnected: understanding memory management informs
your concurrency designs, networking knowledge shapes your API design, database
fundamentals influence your system architecture, and security considerations pervade
every layer.

All material is presented with language-agnostic pseudocode and ASCII diagrams.

---

## Table of Contents

### 1. [Memory Management](memory_management.md)
How programs allocate, use, and release memory. Covers the stack vs. heap distinction,
manual memory management (allocate/free patterns), automatic garbage collection (reference
counting, mark-and-sweep, generational collection), memory leaks, dangling references,
and buffer overflows. Includes ASCII diagrams of memory layouts and GC cycles.

### 2. [Concurrency and Parallelism](concurrency_and_parallelism.md)
Building programs that handle multiple tasks simultaneously. Covers processes vs. threads,
race conditions, mutual exclusion (mutexes, semaphores), deadlock (conditions, prevention,
detection), synchronization primitives, the producer-consumer problem, thread pools, and
the actor model. Includes ASCII timing diagrams of concurrent execution scenarios.

### 3. [Database Fundamentals](database_fundamentals.md)
How software systems store and query persistent data. Covers relational databases (tables,
keys, normalization, SQL operations), ACID transactions, indexing strategies, query
optimization, NoSQL categories (document, key-value, column-family, graph), and the
CAP theorem. Includes ASCII entity-relationship diagrams and normalization examples.

### 4. [Networking Fundamentals](networking_fundamentals.md)
How software communicates across machines. Covers the OSI and TCP/IP models, TCP vs. UDP,
DNS resolution, HTTP request/response cycles, TLS/SSL handshakes, sockets, REST
principles, and common network debugging techniques. Includes ASCII protocol diagrams
and packet flow illustrations.

### 5. [API Design Principles](api_design_principles.md)
How to design clean, consistent, and evolvable interfaces between software components.
Covers REST API design, resource naming, HTTP method semantics, status codes, versioning
strategies, pagination, rate limiting, authentication schemes, and documentation practices.
Includes request/response pseudocode examples and API contract diagrams.

### 6. [Security Fundamentals](security_fundamentals.md)
Essential security knowledge for every developer. Covers the CIA triad (confidentiality,
integrity, availability), authentication vs. authorization, common vulnerabilities (injection,
XSS, CSRF, broken access control), cryptographic primitives (hashing, encryption, digital
signatures), secure coding practices, and the OWASP Top Ten. Includes attack-flow ASCII
diagrams.

### 7. [System Design Basics](system_design_basics.md)
The art and science of building scalable, reliable software systems. Covers load balancing,
caching strategies, horizontal vs. vertical scaling, database sharding and replication,
message queues, CDNs, microservice decomposition, and system design methodology. Includes
ASCII architecture diagrams for common system patterns.

---

## Recommended Reading Order

These topics are relatively independent, but the following order provides the most natural
conceptual build-up:

1. **Memory Management** -- Understanding memory is foundational to everything that follows.
2. **Concurrency and Parallelism** -- Builds directly on memory concepts (shared state, synchronization).
3. **Database Fundamentals** -- How persistent data is stored and queried by applications.
4. **Networking Fundamentals** -- How distributed components communicate.
5. **API Design Principles** -- Designing the interfaces that operate over networks.
6. **Security Fundamentals** -- Securing every layer (memory, concurrency, data, network, API).
7. **System Design Basics** -- Synthesizes all topics into holistic system architecture.

Alternatively, if you are preparing for system design discussions, start with topic 7 and
work backwards into the supporting topics as needed.

## Prerequisites

- All of [Sections 1-4](../01_core_programming/) are strongly recommended.
- [Section 5: Architectural Patterns](../05_architectural_patterns/) provides useful context
  for System Design Basics.
- These are genuinely advanced topics; foundational knowledge makes them far more accessible.

## What You Will Learn

After completing this section, you will be able to:
- Explain how memory is managed at the system level and avoid common memory-related bugs
- Design concurrent programs that avoid race conditions and deadlocks
- Choose appropriate database technologies and design schemas for different use cases
- Understand network protocols and how data moves between systems
- Design clean, consistent APIs that are easy to use and evolve
- Apply fundamental security practices throughout the development lifecycle
- Approach system design problems with a structured methodology
