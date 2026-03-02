# System Design Basics

## Overview

System design is the art and science of building software systems that are scalable,
reliable, and maintainable. As applications grow from serving a handful of users to millions,
the architecture must evolve to handle increased load, ensure availability, and manage data
consistency. System design requires understanding trade-offs -- there is no single correct
architecture, only architectures that are appropriate for given requirements and constraints.

The key dimensions of system design are scalability (handling growing load), reliability
(continuing to work correctly even when components fail), performance (responding quickly),
and maintainability (making the system easy to understand, extend, and operate). These
dimensions often conflict: optimizing for one may compromise another.

This chapter covers the foundational building blocks of distributed systems: load balancing,
caching, scaling strategies, the CAP theorem, database replication and sharding, message
queues, CDNs, and resilience patterns like circuit breakers.

## Key Concepts

### Load Balancing

A load balancer distributes incoming requests across multiple backend servers to prevent
any single server from becoming a bottleneck.

```
                    +-------------------+
  Clients -------> |  Load Balancer    |
                    +-------------------+
                     /       |       \
                    v        v        v
               +--------+ +--------+ +--------+
               |Server 1| |Server 2| |Server 3|
               +--------+ +--------+ +--------+
```

#### Round-Robin

Requests are distributed sequentially across servers.

```
  Request 1 --> Server 1
  Request 2 --> Server 2
  Request 3 --> Server 3
  Request 4 --> Server 1  (cycles back)
  Request 5 --> Server 2
```

Simple and fair when servers have equal capacity and requests have equal cost.

#### Least Connections

Route each request to the server with the fewest active connections.

```
  Server 1: 12 active connections
  Server 2: 3 active connections   <-- Next request goes here
  Server 3: 8 active connections
```

Better for variable-length requests (some fast, some slow).

#### Consistent Hashing

Maps both servers and requests onto a hash ring. Each request is routed to the nearest
server clockwise on the ring. When a server is added or removed, only a fraction of
requests are redistributed.

```
                    0
                    |
         Server A  *         * Server B
                /     \
              /         \
            /             \
  270 ----*                 *---- 90
            \             /
              \         /
                \     /
         Server D  *         * Server C
                    |
                   180

  Request hash = 45 --> routes to Server B (next clockwise)
  Request hash = 200 --> routes to Server D (next clockwise)
```

### Caching Strategies

Caching stores frequently accessed data closer to the consumer, dramatically reducing
latency and backend load.

```
  Client --> Cache --[HIT]--> Return cached data (fast)
                  \--[MISS]--> Backend --> Store in cache --> Return data
```

#### Cache-Aside (Lazy Loading)

```
FUNCTION get_user(user_id: String) -> User:
    SET cached = LOOKUP user_id IN cache
    IF cached IS NOT NULL THEN
        RETURN cached                    // Cache hit
    END IF

    SET user = QUERY database FOR user_id  // Cache miss
    STORE user_id -> user IN cache WITH ttl = 300 SECONDS
    RETURN user
END FUNCTION
```

#### Write-Through

Every write goes to both the cache and the database simultaneously.

```
FUNCTION update_user(user_id: String, data: Map):
    SET user = UPDATE database SET data WHERE id = user_id
    STORE user_id -> user IN cache       // Cache always up to date
    RETURN user
END FUNCTION
```

#### Write-Back (Write-Behind)

Writes go to the cache first; the cache asynchronously flushes to the database.

```
  Client --> Cache (write) --> ACK (fast)
                |
                +-- async --> Database (eventual write)

  Pros: Very fast writes
  Cons: Risk of data loss if cache fails before flush
```

#### Write-Around

Writes go directly to the database, bypassing the cache. The cache is populated only on reads.

```
  Write: Client --> Database (cache not updated)
  Read:  Client --> Cache (miss) --> Database --> Cache (populated)
```

### Horizontal vs Vertical Scaling

```
  VERTICAL SCALING                    HORIZONTAL SCALING
  (Scale Up)                          (Scale Out)

  Before:        After:               Before:        After:
  +--------+     +----------+         +--------+     +----+ +----+ +----+
  | Server |     | BIGGER   |         | Server |     | S1 | | S2 | | S3 |
  | 4 CPU  |     | SERVER   |         +--------+     +----+ +----+ +----+
  | 16 GB  |     | 32 CPU   |
  +--------+     | 256 GB   |
                 +----------+

  Limits: Hardware ceiling   Limits: Complexity of distribution
  Cost: Expensive hardware   Cost: More machines, but commodity
```

| Aspect             | Vertical Scaling       | Horizontal Scaling         |
|--------------------|------------------------|----------------------------|
| Approach           | Bigger machine         | More machines              |
| Complexity         | Low (single node)      | High (distributed system)  |
| Ceiling            | Hardware limits        | Nearly unlimited           |
| Downtime           | Often required         | Zero-downtime possible     |
| Cost efficiency    | Diminishing returns    | Linear scaling             |
| Data consistency   | Simple (one node)      | Complex (distributed)      |

### CAP Theorem

In a distributed system, you can guarantee at most two of three properties:

```
             Consistency
                /\
               /  \
              /    \
             / CP   \
            /  zone  \
           /          \
          /     CA     \
         /    zone     \
        +--------------+
       /                \
      / AP zone          \
     /                    \
    Availability ---- Partition Tolerance

  CA: Consistent + Available (only possible without network partitions)
  CP: Consistent + Partition-tolerant (may sacrifice availability)
  AP: Available + Partition-tolerant (may return stale data)
```

- **Consistency**: Every read receives the most recent write
- **Availability**: Every request receives a response (not necessarily the latest data)
- **Partition Tolerance**: System continues operating despite network partitions

In practice, network partitions are inevitable, so the real choice is between CP and AP.

### Database Replication

#### Master-Slave (Primary-Replica)

```
  +----------+     replication     +----------+
  | Primary  | ------------------> | Replica 1|  (read-only)
  | (writes) | ------------------> | Replica 2|  (read-only)
  +----------+                     +----------+

  Writes --> Primary only
  Reads  --> Any replica (or primary)
```

```
FUNCTION write_data(key: String, value: Any):
    WRITE TO primary_database(key, value)
    // Replicas receive update asynchronously (or synchronously)
END FUNCTION

FUNCTION read_data(key: String) -> Any:
    SET replica = SELECT_REPLICA(replicas)
    RETURN READ FROM replica(key)
END FUNCTION
```

#### Multi-Master

```
  +----------+  <-- replication -->  +----------+
  | Master 1 |                      | Master 2 |
  | (R/W)    |  <-- replication -->  | (R/W)    |
  +----------+                      +----------+

  Both accept writes --> conflict resolution required
```

### Database Sharding

Distributes data across multiple databases based on a shard key.

```
  Shard Key: user_id

  +------------------+  +------------------+  +------------------+
  | Shard 1          |  | Shard 2          |  | Shard 3          |
  | user_id 1-1000   |  | user_id 1001-2000|  | user_id 2001-3000|
  +------------------+  +------------------+  +------------------+

  FUNCTION get_shard(user_id: Integer) -> Shard:
      SET shard_number = user_id MOD number_of_shards
      RETURN shards[shard_number]
  END FUNCTION
```

```
  Sharding Strategies:
  +------------------+--------------------------------------------+
  | Range-based      | Shard by ranges (A-M, N-Z)                |
  | Hash-based       | Shard by hash(key) MOD N                   |
  | Directory-based  | Lookup table maps keys to shards           |
  +------------------+--------------------------------------------+

  Challenges:
  - Cross-shard queries are expensive
  - Rebalancing when adding shards is complex
  - Some shard keys create hot spots
```

### Message Queues

Decouple producers from consumers, enabling asynchronous processing and load smoothing.

```
  +----------+    +--------------------+    +----------+
  | Producer | -> | Message Queue      | -> | Consumer |
  | Service  |    | [msg1|msg2|msg3|..]|    | Service  |
  +----------+    +--------------------+    +----------+
                         |
                         +---> | Consumer |
                               | Service  |
                               +----------+

  Benefits:
  - Producer does not wait for consumer
  - Consumers process at their own pace
  - Handles traffic spikes (queue absorbs bursts)
  - Failed consumers can retry from queue
```

```
FUNCTION handle_order(order: Order):
    // Validate and save order (synchronous)
    SAVE order TO database

    // Offload slow processing to queue (asynchronous)
    PUBLISH TO "order_processing" QUEUE: {
        order_id: order.id,
        action: "process_payment"
    }
    PUBLISH TO "notification" QUEUE: {
        user_id: order.user_id,
        message: "Order received"
    }

    RETURN { status: "accepted", order_id: order.id }
END FUNCTION

// Separate consumer processes these asynchronously
FUNCTION payment_consumer():
    LOOP
        SET message = CONSUME FROM "order_processing" QUEUE
        process_payment(message.order_id)
        ACKNOWLEDGE message
    END LOOP
END FUNCTION
```

### CDN (Content Delivery Network)

```
                        Origin Server
                        (San Francisco)
                             |
              +--------------+--------------+
              |              |              |
         +--------+    +--------+    +--------+
         |  CDN   |    |  CDN   |    |  CDN   |
         | Edge   |    | Edge   |    | Edge   |
         | Tokyo  |    | London |    | Sydney |
         +--------+    +--------+    +--------+
              ^              ^              ^
              |              |              |
         Users in       Users in       Users in
         Asia           Europe         Australia

  User request --> nearest CDN edge
    Cache HIT  --> serve directly (fast, <50ms)
    Cache MISS --> fetch from origin, cache, serve
```

### Circuit Breaker Pattern

Prevents cascading failures by stopping calls to a failing service.

```
  States:
  +--------+   failure threshold   +---------+   timeout   +-----------+
  | CLOSED | --------------------> | OPEN    | ----------> | HALF-OPEN |
  | (normal|                       | (reject |             | (test one |
  |  flow) | <-------------------- |  calls) | <---------- |  request) |
  +--------+   success in          +---------+   failure   +-----------+
               half-open
```

```
FUNCTION create_circuit_breaker(threshold: Integer, timeout: Duration) -> Breaker:
    SET breaker.state = "CLOSED"
    SET breaker.failure_count = 0
    SET breaker.threshold = threshold
    SET breaker.timeout = timeout
    SET breaker.last_failure_time = NULL
    RETURN breaker
END FUNCTION

FUNCTION call_with_breaker(breaker: Breaker, operation: Function) -> Result:
    IF breaker.state == "OPEN" THEN
        IF CURRENT_TIME() - breaker.last_failure_time > breaker.timeout THEN
            SET breaker.state = "HALF_OPEN"
        ELSE
            RAISE CircuitOpenError("Service unavailable")
        END IF
    END IF

    TRY
        SET result = operation()
        // Success
        IF breaker.state == "HALF_OPEN" THEN
            SET breaker.state = "CLOSED"
        END IF
        SET breaker.failure_count = 0
        RETURN result
    CATCH error
        SET breaker.failure_count = breaker.failure_count + 1
        SET breaker.last_failure_time = CURRENT_TIME()
        IF breaker.failure_count >= breaker.threshold THEN
            SET breaker.state = "OPEN"
        END IF
        RAISE error
    END TRY
END FUNCTION
```

## Visual Representation: Complete System Architecture

```
  +--------+    +--------+    +------+    +------------------+
  | Client | -> |  CDN   | -> | Load | -> | App Server 1     |
  +--------+    +--------+    | Bal. | -> | App Server 2     | -> +-------+
                              +------+ -> | App Server 3     |    | Cache |
                                          +------------------+    +-------+
                                                  |                   |
                                          +-------+-------+          |
                                          |               |          |
                                     +--------+    +----------+     |
                                     |  Queue  |   | Primary  |<----+
                                     +--------+    | Database |
                                          |        +----------+
                                     +--------+        |
                                     |Workers |   +----------+
                                     +--------+   | Replicas |
                                                  +----------+
```

## Common Pitfalls and Best Practices

| Pitfall                           | Best Practice                                   |
|-----------------------------------|--------------------------------------------------|
| Premature optimization            | Design for current scale; plan for 10x            |
| Single point of failure           | Redundancy at every layer                         |
| Ignoring cache invalidation       | Use TTLs and explicit invalidation strategies     |
| Unbounded queues                  | Set queue size limits; implement backpressure     |
| Choosing sharding too early       | Start with replication; shard when necessary       |
| Synchronous inter-service calls   | Use async messaging for non-critical paths        |
| No circuit breakers               | Wrap all external service calls with breakers     |

## Real-World Applications

- **Social media feeds**: Caching, CDN, fan-out on write vs fan-out on read
- **E-commerce checkout**: Message queues for order processing, circuit breakers for payment
- **Video streaming**: CDN edge caching, adaptive bitrate, sharded metadata stores
- **Chat applications**: WebSocket connections, message queues, presence tracking
- **Search engines**: Sharded indexes, replicated query servers, CDN for static assets

## Related Topics

- [Database Fundamentals](database_fundamentals.md) -- Replication and sharding details
- [Networking Fundamentals](networking_fundamentals.md) -- HTTP, DNS, and CDN internals
- [Concurrency and Parallelism](concurrency_and_parallelism.md) -- Async processing patterns
- [API Design Principles](api_design_principles.md) -- API gateway and rate limiting

## Practice Problems

1. Design a URL shortening service. Address storage, redirection, analytics, and how to
   handle billions of URLs.

2. Given a system that currently handles 100 requests per second, design a scaling strategy
   to support 100,000 requests per second. Identify bottlenecks at each stage.

3. Implement a cache-aside pattern with TTL-based expiration and a cache invalidation
   mechanism triggered by database writes.

4. Design a notification system that sends emails, push notifications, and SMS. Use message
   queues to decouple the sending from the triggering events.

5. Explain how you would shard a user database with 500 million users. Choose a shard key,
   explain the strategy, and discuss how to handle cross-shard queries.

6. Implement a circuit breaker that tracks failure rates over a sliding window rather than
   a simple counter.

7. For a global e-commerce platform, design a CDN strategy for static assets, product images,
   and API responses. Explain invalidation for each.
