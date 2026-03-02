# Distributed Systems

## Overview

A distributed system is a collection of independent machines that appear to
users as a single coherent system. They communicate by passing messages over a
network to coordinate their actions.

### Why Distributed Systems Are Necessary

- **Scale**: A single machine has finite CPU, memory, and storage.
- **Availability**: Hardware fails. Multiple machines keep the system running.
- **Latency**: Geographically nearby nodes reduce round-trip time.
- **Isolation**: Separating concerns limits the blast radius of failures.

### Why Distributed Systems Are Fundamentally Hard

- Messages can be lost, delayed, duplicated, or reordered.
- Clocks drift and cannot be perfectly synchronized.
- Any node can fail at any moment, including mid-operation.
- There is no shared memory; all coordination requires explicit communication.
- The CAP theorem: during a network partition, a system must choose between
  consistency and availability.

---

## Consensus Algorithms

Consensus is getting multiple nodes to agree on a single value even when some
nodes or messages fail.

### Paxos (High-Level Phases)

**Phase 1 -- Prepare**: A proposer selects proposal number N and sends
PREPARE(N) to a majority of acceptors. Each acceptor that has not responded to
a higher number replies with a PROMISE and any previously accepted value.

**Phase 2 -- Accept**: If the proposer receives a majority of promises, it
sends ACCEPT(N, value). Acceptors that have not promised a higher number accept.
Once a majority accept, the value is chosen.

```
  Proposer          Acceptor A     Acceptor B     Acceptor C
     |-- PREPARE(1) -->|              |              |
     |-- PREPARE(1) ------------->    |              |
     |-- PREPARE(1) ------------------------------>  |
     |<- PROMISE(1) --|              |              |
     |<- PROMISE(1) -------------|              |
     |<- PROMISE(1) ------------------------------|
     |-- ACCEPT(1,v) ->|              |              |
     |-- ACCEPT(1,v) ------------>    |              |
     |-- ACCEPT(1,v) ----------------------------->  |
     v  Value v chosen (majority accepted)           v
```

### Raft (Leader Election, Log Replication, Safety)

Raft decomposes consensus into three sub-problems.

#### Leader Election

```
  +----------+    timeout    +----------+   majority    +----------+
  | Follower | -----------> | Candidate | -- votes --> |  Leader  |
  +----------+              +----------+               +----------+
       ^                         |                           |
       |    higher term found    |    higher term found      |
       +-------------------------+---------------------------+
```

1. All nodes start as followers with randomized election timeouts.
2. A follower that times out becomes a candidate, increments its term, votes
   for itself, and requests votes from all other nodes.
3. A node grants its vote if it has not voted in this term and the candidate's
   log is at least as up-to-date as its own.
4. A candidate receiving a majority of votes becomes the leader.

```
FUNCTION REQUEST_VOTE(candidate, term, lastLogIndex, lastLogTerm)
    IF term < currentTerm THEN
        RETURN (currentTerm, FALSE)
    END IF
    IF votedFor IS NULL OR votedFor = candidate THEN
        IF lastLogTerm > myLastLogTerm THEN
            SET votedFor TO candidate
            RETURN (currentTerm, TRUE)
        END IF
        IF lastLogTerm = myLastLogTerm AND lastLogIndex >= myLastLogIndex THEN
            SET votedFor TO candidate
            RETURN (currentTerm, TRUE)
        END IF
    END IF
    RETURN (currentTerm, FALSE)
END FUNCTION
```

#### Log Replication

The leader appends client entries to its log, sends APPEND_ENTRIES to followers,
and commits once a majority acknowledge. Followers learn of commits via
subsequent heartbeats.

#### Safety Properties

- **Election Safety**: At most one leader per term.
- **Leader Append-Only**: A leader never overwrites its own log entries.
- **Log Matching**: Same index and term implies all preceding entries match.
- **Leader Completeness**: Committed entries appear in all future leaders' logs.

---

## Distributed Transactions

### Two-Phase Commit (2PC)

Ensures a transaction commits on all participants or aborts on all of them.

```
  Coordinator              Participant A      Participant B
      |--- PREPARE ---------->|                    |
      |--- PREPARE ------------------------------>|
      |<-- VOTE YES/NO ------|                    |
      |<-- VOTE YES/NO ----------------------------|
      |  (all YES? COMMIT : ABORT)                 |
      |--- COMMIT/ABORT ----->|                    |
      |--- COMMIT/ABORT ---------------------------->|
      |<-- ACK ---------------|                    |
      |<-- ACK ----------------------------------|
```

```
FUNCTION TWO_PHASE_COMMIT(coordinator, participants, transaction)
    SET votes TO EMPTY LIST
    FOR EACH participant IN participants DO
        APPEND SEND_PREPARE(participant, transaction) TO votes
    END FOR
    IF ALL votes ARE "YES" THEN
        FOR EACH participant IN participants DO
            SEND_COMMIT(participant, transaction)
        END FOR
        RETURN "COMMITTED"
    ELSE
        FOR EACH participant IN participants DO
            SEND_ABORT(participant, transaction)
        END FOR
        RETURN "ABORTED"
    END IF
END FUNCTION
```

**Limitation**: If the coordinator crashes after PREPARE but before COMMIT/ABORT,
participants block until it recovers.

### Three-Phase Commit (3PC)

Adds a PRE-COMMIT phase to reduce blocking. If all vote YES, the coordinator
sends PRE-COMMIT before the final COMMIT. This lets participants recover
independently -- any participant that received PRE-COMMIT knows the decision was
to commit.

### Saga Pattern

Sagas break long-lived transactions into local transactions, each with a
compensating action for rollback.

#### Choreography (Event-Driven)

Each service listens for events and decides independently what to do next.

```
  Service A        Event Bus         Service B        Service C
     |-- OrderCreated ->|                |                |
     |                  |-- OrderCreated -->|              |
     |                  |<- PaymentDone ---|              |
     |                  |-- PaymentDone ----------------->|
     |                  |<- ShipmentDone -----------------|
     |<- ShipmentDone --|                |                |
```

```
FUNCTION SAGA_CHOREOGRAPHY_HANDLER(service, event)
    IF event.type = "ORDER_CREATED" AND service = "PaymentService" THEN
        SET result TO PROCESS_PAYMENT(event.orderId)
        IF result = SUCCESS THEN
            PUBLISH("PAYMENT_COMPLETED", event.orderId)
        ELSE
            PUBLISH("PAYMENT_FAILED", event.orderId)
        END IF
    END IF
    IF event.type = "PAYMENT_COMPLETED" AND service = "ShippingService" THEN
        SET result TO ARRANGE_SHIPMENT(event.orderId)
        IF result = SUCCESS THEN
            PUBLISH("SHIPMENT_ARRANGED", event.orderId)
        ELSE
            PUBLISH("SHIPMENT_FAILED", event.orderId)
        END IF
    END IF
    IF event.type = "SHIPMENT_FAILED" AND service = "PaymentService" THEN
        REVERSE_PAYMENT(event.orderId)
        PUBLISH("PAYMENT_REVERSED", event.orderId)
    END IF
    IF event.type = "PAYMENT_FAILED" AND service = "OrderService" THEN
        CANCEL_ORDER(event.orderId)
    END IF
END FUNCTION
```

#### Orchestration (Central Coordinator)

A central orchestrator directs each step and handles compensation on failure.

```
FUNCTION SAGA_ORCHESTRATOR(orderId)
    SET completedSteps TO EMPTY LIST

    SET steps TO ["OrderService:CREATE", "PaymentService:CHARGE", "ShippingService:SHIP"]
    FOR EACH step IN steps DO
        SET result TO CALL_SERVICE(step, orderId)
        IF result = FAILURE THEN
            COMPENSATE(completedSteps, orderId)
            RETURN "SAGA_FAILED"
        END IF
        APPEND step TO completedSteps
    END FOR
    RETURN "SAGA_COMPLETED"
END FUNCTION

FUNCTION COMPENSATE(completedSteps, orderId)
    FOR EACH step IN REVERSE(completedSteps) DO
        IF step = "PaymentService:CHARGE" THEN
            CALL_SERVICE("PaymentService", "REFUND", orderId)
        END IF
        IF step = "OrderService:CREATE" THEN
            CALL_SERVICE("OrderService", "CANCEL_ORDER", orderId)
        END IF
    END FOR
END FUNCTION
```

---

## Consistency Models

### Strong Consistency

Every read returns the most recent write. All nodes see the same data at the
same time. Simplest to reason about but most expensive.

```
  Time --->
  Client A:   WRITE(x=5) -----+
  Server:     x=0 ... x=5 ----+---->
  Client B:             READ(x) = 5    (always sees latest write)
```

### Eventual Consistency

If no new updates are made, all replicas eventually converge. Reads may return
stale data in the interim.

```
  Time --->
  Client A:   WRITE(x=5) ------+
  Replica 1:  x=5  (immediate) |
  Replica 2:  x=0  ... x=0 ... x=5  (delay)
  Replica 3:  x=0  ... x=5          (delay)
  Client B:   READ from Replica 2 = 0  (stale!)
  Client B:   READ from Replica 2 = 5  (later, converged)
```

### Causal Consistency

If operation A causally precedes B, every node sees A before B. Concurrent
operations (no causal link) may appear in different orders on different nodes.

```
  Time --->
  Client A:   WRITE(x=1) -------->  WRITE(y=2, after reading x=1)
  Client B:   sees x=1 first, then y=2       (causal order preserved)

  Concurrent, unrelated writes:
  Client D:   WRITE(a=10)
  Client E:   WRITE(b=20)
  Client F:   may see either order            (both valid)
```

---

## CRDTs: Conflict-Free Replicated Data Types

CRDTs are data structures where concurrent updates on different replicas can
always be merged automatically without conflicts.

### G-Counter (Grow-Only Counter)

Each node maintains its own count. The global value is the sum of all per-node
counts. Merging takes the element-wise maximum.

```
  Node A: [A:3, B:0, C:0]   total = 3
  Node B: [A:0, B:5, C:0]   total = 5
  Node C: [A:0, B:0, C:2]   total = 2
  Merged: [A:3, B:5, C:2]   total = 10
```

```
FUNCTION G_COUNTER_INCREMENT(counter, nodeId)
    SET counter[nodeId] TO counter[nodeId] + 1
END FUNCTION

FUNCTION G_COUNTER_VALUE(counter)
    SET total TO 0
    FOR EACH nodeId IN counter DO
        SET total TO total + counter[nodeId]
    END FOR
    RETURN total
END FUNCTION

FUNCTION G_COUNTER_MERGE(counterA, counterB)
    SET merged TO EMPTY MAP
    FOR EACH nodeId IN UNION(KEYS(counterA), KEYS(counterB)) DO
        SET merged[nodeId] TO MAX(counterA[nodeId], counterB[nodeId])
    END FOR
    RETURN merged
END FUNCTION
```

### PN-Counter (Positive-Negative Counter)

Uses two G-Counters: P for increments, N for decrements. Value = P - N.

```
FUNCTION PN_COUNTER_INCREMENT(pCounter, nodeId)
    G_COUNTER_INCREMENT(pCounter, nodeId)
END FUNCTION

FUNCTION PN_COUNTER_DECREMENT(nCounter, nodeId)
    G_COUNTER_INCREMENT(nCounter, nodeId)
END FUNCTION

FUNCTION PN_COUNTER_VALUE(pCounter, nCounter)
    RETURN G_COUNTER_VALUE(pCounter) - G_COUNTER_VALUE(nCounter)
END FUNCTION
```

---

## Vector Clocks

Vector clocks track causality across nodes. Each node maintains a vector of
logical timestamps, one entry per node.

**Rules:**
1. Before sending a message, a node increments its own entry.
2. On receiving, take the element-wise maximum of both vectors, then increment
   your own entry.

### Step-by-Step Trace

Three nodes (A, B, C), initial vectors all [0, 0, 0]:

```
  Step  Action                  A's clock    B's clock    C's clock
  ----  ----------------------  -----------  -----------  -----------
   1    A does local work       [1, 0, 0]
   2    A sends msg to B        [2, 0, 0]
   3    B receives from A                    [2, 1, 0]
   4    B sends msg to C                     [2, 2, 0]
   5    C receives from B                                 [2, 2, 1]
   6    A does local work       [3, 0, 0]
   7    C sends msg to A                                  [2, 2, 2]
   8    A receives from C       [4, 2, 2]

  Comparing events:
  - Step 1 [1,0,0] HAPPENS-BEFORE Step 3 [2,1,0]  (all <= and one <)
  - Step 6 [3,0,0] CONCURRENT with Step 5 [2,2,1] (3>2 but 0<2)
```

```
FUNCTION VECTOR_CLOCK_INCREMENT(clock, nodeId)
    SET clock[nodeId] TO clock[nodeId] + 1
    RETURN clock
END FUNCTION

FUNCTION VECTOR_CLOCK_MERGE(clockA, clockB)
    SET merged TO EMPTY MAP
    FOR EACH nodeId IN UNION(KEYS(clockA), KEYS(clockB)) DO
        SET merged[nodeId] TO MAX(clockA[nodeId], clockB[nodeId])
    END FOR
    RETURN merged
END FUNCTION

FUNCTION HAPPENS_BEFORE(clockA, clockB)
    SET atLeastOneLess TO FALSE
    FOR EACH nodeId IN UNION(KEYS(clockA), KEYS(clockB)) DO
        IF clockA[nodeId] > clockB[nodeId] THEN
            RETURN FALSE
        END IF
        IF clockA[nodeId] < clockB[nodeId] THEN
            SET atLeastOneLess TO TRUE
        END IF
    END FOR
    RETURN atLeastOneLess
END FUNCTION
```

---

## Partition Strategies and Consistent Hashing

When a dataset exceeds a single node's capacity, it must be partitioned.

- **Range Partitioning**: Contiguous key ranges per node. Simple but prone to
  hotspots.
- **Hash Partitioning**: Hash the key to pick a node. Even distribution but
  loses range query locality.
- **Consistent Hashing**: Keys and nodes are mapped onto a ring. Each key is
  assigned to the first node clockwise. When a node leaves, only its keys
  redistribute -- other assignments are unaffected.

```
          Node A
           /    \
         /        \
    Key k1      Node D
       |            |
    Node C      Key k3
         \        /
           \    /
          Key k2
          Node B

  Removing Node C only moves keys between B and C
  to the next clockwise node. All others stay put.
```

For deeper coverage including virtual nodes, see **system_design_basics.md**.

---

## Byzantine Fault Tolerance

Standard consensus assumes nodes may crash but never lie. BFT handles nodes
that behave arbitrarily -- lying, sending contradictory messages, or colluding.
Tolerating f Byzantine faults requires 3f + 1 nodes. Practical BFT (PBFT) uses
three rounds with O(n^2) message complexity. Essential in adversarial
environments but too expensive when crash-fault tolerance suffices.

---

## Fallacies of Distributed Computing

Eight false assumptions newcomers commonly make:

1. **The network is reliable.** Plan for retries, timeouts, idempotency.
2. **Latency is zero.** Network calls are far slower than local calls.
3. **Bandwidth is infinite.** Large payloads create bottlenecks.
4. **The network is secure.** Data in transit can be intercepted.
5. **Topology does not change.** Nodes join, leave, and move.
6. **There is one administrator.** Different orgs manage different parts.
7. **Transport cost is zero.** Serialization and transfer cost resources.
8. **The network is homogeneous.** Nodes vary in hardware and software.

---

## Practice Problems

1. **Consensus Under Failure**: Three nodes participate in Raft leader election.
   Node A has term 4 with log index 7. Node B has term 4 with log index 6.
   Node C has term 3 with log index 8. Node A starts an election for term 5.
   Which nodes grant their vote to A, and why? Does A become leader?

2. **2PC Failure Scenario**: A coordinator sends PREPARE to three participants.
   Participants 1 and 2 vote YES. Participant 3 crashes before voting. The
   coordinator times out. Trace the protocol. What happens when Participant 3
   recovers?

3. **Saga Compensation Design**: Design a saga for a travel booking (flight,
   hotel, car rental). Write pseudocode for the orchestrator and compensation.
   What happens if car rental fails after flight and hotel succeed?

4. **CRDT Merge Exercise**: Three nodes independently modify a G-Counter.
   Node X: [X:4, Y:0, Z:0]. Node Y: [X:2, Y:3, Z:0]. Node Z: [X:2, Y:1, Z:5].
   Compute the merged counter and its total value.

5. **Vector Clock Causality**: Determine which pairs are causally ordered and
   which are concurrent:
   - Event P: [2, 0, 0]
   - Event Q: [1, 2, 0]
   - Event R: [2, 2, 1]
   - Event S: [0, 0, 3]

6. **Partition Rebalancing**: A consistent hashing ring has 4 nodes (A, B, C, D)
   each with 3 virtual nodes. Node C fails permanently. Describe which keys
   move and where. Then Node E joins. Explain the rebalancing and why consistent
   hashing minimizes data movement compared to hash-mod-N.
