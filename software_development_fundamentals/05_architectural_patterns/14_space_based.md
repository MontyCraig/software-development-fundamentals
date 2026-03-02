# Space-Based Architecture

## Overview and Intent

Space-Based Architecture (SBA) addresses the scalability limitations of
traditional centralized database systems by distributing both processing and
data across multiple nodes using in-memory data grids. Instead of routing all
operations through a single database, each processing unit holds a copy of the
data it needs in memory and synchronizes changes with other units through a
messaging grid.

The intent is to achieve near-linear scalability by eliminating the database
as a bottleneck. In traditional architectures, the database becomes the
constraint as load increases -- connection pools fill, locks contend, and query
latency rises. SBA removes this constraint by replicating data into memory
across processing units, enabling each unit to serve requests independently.

The name comes from "tuple space," a shared memory paradigm where processes
communicate by writing and reading tuples in a distributed space. SBA applies
this concept at the architectural level, using in-memory data grids as the
shared space and processing units as the participants.

## Structure and Components

### Component Diagram

```
                    +-------------------+
                    |   Client Requests |
                    +--------+----------+
                             |
                    +--------+----------+
                    |  VIRTUALIZED      |
                    |  MIDDLEWARE        |
                    |                   |
                    | +---------------+ |
                    | | Messaging     | |
                    | | Grid          | |
                    | +-------+-------+ |
                    | | Data Grid     | |
                    | | (Replication) | |
                    | +-------+-------+ |
                    | | Processing    | |
                    | | Grid          | |
                    | +---------------+ |
                    +---+-----+-----+---+
                        |     |     |
              +---------+     |     +---------+
              |               |               |
    +---------+---+  +--------+----+  +-------+-----+
    |Processing   |  |Processing   |  |Processing   |
    |Unit 1       |  |Unit 2       |  |Unit N       |
    |             |  |             |  |             |
    | +--------+  |  | +--------+  |  | +--------+  |
    | |In-Mem  |  |  | |In-Mem  |  |  | |In-Mem  |  |
    | |Data    |  |  | |Data    |  |  | |Data    |  |
    | |Grid    |  |  | |Grid    |  |  | |Grid    |  |
    | +--------+  |  | +--------+  |  | +--------+  |
    | +--------+  |  | +--------+  |  | +--------+  |
    | |App     |  |  | |App     |  |  | |App     |  |
    | |Logic   |  |  | |Logic   |  |  | |Logic   |  |
    | +--------+  |  | +--------+  |  | +--------+  |
    +------+------+  +------+------+  +------+------+
           |                |                |
           +--------+-------+--------+-------+
                    |                |
           +--------+------+ +------+--------+
           | Data Pumps    | | Data Writers  |
           | (Async read)  | | (Async write) |
           +--------+------+ +------+--------+
                    |                |
           +--------+----------------+-------+
           |         PERSISTENT DATABASE      |
           |         (Eventual backup)        |
           +----------------------------------+
```

### Key Components

1. **Processing Unit**: A self-contained application instance that holds
   both business logic and an in-memory data grid. Each unit can handle
   requests independently.

2. **In-Memory Data Grid**: Replicated, partitioned data store distributed
   across processing units. Provides low-latency read/write access.

3. **Messaging Grid**: Routes messages between processing units and manages
   session affinity, request routing, and inter-unit communication.

4. **Data Grid (Replication)**: Synchronizes data changes between processing
   units to maintain consistency across the grid.

5. **Data Pumps**: Asynchronous processes that read from the in-memory grid
   and write to the persistent database for durability.

6. **Data Writers**: Components that flush in-memory changes to persistent
   storage on a configurable schedule.

## How It Works

```
  Request arrives
       |
       v
  +----+------+
  | Messaging |  1. Route request to a processing unit
  | Grid      |     (load balance or session affinity)
  +----+------+
       |
       v
  +----+------+
  | Processing|  2. Read data from local in-memory grid
  | Unit      |  3. Execute business logic
  +----+------+  4. Write results to local in-memory grid
       |
       v
  +----+------+
  | Data Grid |  5. Replicate changes to other processing units
  | Replicator|     (synchronous or asynchronous)
  +----+------+
       |
  +----+----+-----+
  |         |     |
  v         v     v
 PU 2     PU 3   PU N   6. Other units receive replicated data
                          7. Their in-memory grids are updated

  Asynchronously:
  +----+------+
  | Data Pump |  8. Periodically flush changes to persistent DB
  +----+------+  9. Database serves as backup, not primary store
       |
       v
  [Persistent Database]
```

## Pseudocode Example

```
// === PROCESSING UNIT ===

STRUCTURE ProcessingUnit:
    id: String
    dataGrid: InMemoryDataGrid
    messagingGrid: MessagingGrid
    applicationLogic: ApplicationModule
END STRUCTURE

FUNCTION ProcessingUnit.HandleRequest(request: Request) -> Response:
    // All data access is local -- no remote database call
    IF request.type = "GET_SESSION" THEN
        SET session = this.dataGrid.Get("sessions", request.sessionId)
        IF session IS NULL THEN
            RETURN Response.NotFound("Session not found")
        END IF
        RETURN Response.OK(session)

    ELSE IF request.type = "PLACE_BID" THEN
        RETURN this.PlaceBid(request)

    ELSE IF request.type = "GET_AUCTION" THEN
        SET auction = this.dataGrid.Get("auctions", request.auctionId)
        RETURN Response.OK(auction)
    END IF
END FUNCTION

FUNCTION ProcessingUnit.PlaceBid(request: BidRequest) -> Response:
    // Lock the auction entry in the data grid
    SET lock = this.dataGrid.Lock("auctions", request.auctionId)

    TRY
        SET auction = this.dataGrid.Get("auctions", request.auctionId)

        IF auction IS NULL THEN
            RETURN Response.NotFound("Auction not found")
        END IF
        IF auction.status != "ACTIVE" THEN
            RETURN Response.BadRequest("Auction is not active")
        END IF
        IF request.amount <= auction.currentBid THEN
            RETURN Response.BadRequest("Bid must exceed current bid of " + auction.currentBid)
        END IF

        // Update auction in local grid
        SET auction.currentBid = request.amount
        SET auction.highestBidderId = request.bidderId
        SET auction.bidCount = auction.bidCount + 1
        SET auction.lastBidAt = Now()

        // Write to local in-memory grid
        this.dataGrid.Put("auctions", request.auctionId, auction)

        // Record bid history
        SET bid = {
            id: GenerateId(),
            auctionId: request.auctionId,
            bidderId: request.bidderId,
            amount: request.amount,
            timestamp: Now()
        }
        this.dataGrid.Put("bids", bid.id, bid)

        // Data grid replication happens automatically to other PUs

        RETURN Response.OK({
            bidId: bid.id,
            newHighBid: auction.currentBid,
            bidCount: auction.bidCount
        })

    FINALLY
        lock.Release()
    END TRY
END FUNCTION

// === IN-MEMORY DATA GRID ===

STRUCTURE InMemoryDataGrid:
    partitions: Map of String to DataPartition
    replicationMode: String   // "SYNC" or "ASYNC"
    replicator: DataReplicator
END STRUCTURE

FUNCTION InMemoryDataGrid.Get(space: String, key: String) -> Any:
    SET partition = this.partitions[space]
    IF partition IS NULL THEN RETURN NULL END IF
    RETURN partition.Get(key)
END FUNCTION

FUNCTION InMemoryDataGrid.Put(space: String, key: String, value: Any):
    SET partition = this.partitions[space]
    IF partition IS NULL THEN
        SET partition = NEW DataPartition(space)
        SET this.partitions[space] = partition
    END IF

    partition.Put(key, value)

    // Trigger replication
    IF this.replicationMode = "SYNC" THEN
        this.replicator.ReplicateSync(space, key, value)
    ELSE
        this.replicator.ReplicateAsync(space, key, value)
    END IF
END FUNCTION

FUNCTION InMemoryDataGrid.Lock(space: String, key: String) -> Lock:
    // Distributed lock across all processing units
    SET lockKey = space + ":" + key
    SET lock = this.replicator.AcquireDistributedLock(lockKey, timeout: 5000)
    IF lock IS NULL THEN
        RAISE LockTimeoutError("Could not acquire lock for " + lockKey)
    END IF
    RETURN lock
END FUNCTION

// === DATA PUMP (Async Persistence) ===

STRUCTURE DataPump:
    dataGrid: InMemoryDataGrid
    database: PersistentDatabase
    batchSize: Integer
    flushInterval: Duration
END STRUCTURE

FUNCTION DataPump.Start():
    LOOP FOREVER DO
        Sleep(this.flushInterval)
        this.FlushChanges()
    END LOOP
END FUNCTION

FUNCTION DataPump.FlushChanges():
    SET changes = this.dataGrid.GetPendingChanges(limit: this.batchSize)

    IF Length(changes) = 0 THEN RETURN END IF

    SET batch = this.database.BeginBatch()
    FOR EACH change IN changes DO
        IF change.type = "PUT" THEN
            batch.Upsert(change.space, change.key, change.value)
        ELSE IF change.type = "DELETE" THEN
            batch.Delete(change.space, change.key)
        END IF
    END FOR

    TRY
        batch.Commit()
        this.dataGrid.AcknowledgeChanges(changes)
        LOG "Flushed " + Length(changes) + " changes to database"
    CATCH error
        LOG "Flush failed: " + error.message + ". Will retry."
        batch.Rollback()
    END TRY
END FUNCTION

// === MESSAGING GRID ===

FUNCTION MessagingGrid.Route(request: Request) -> ProcessingUnit:
    // Session affinity: route to the PU that has the session
    IF request.sessionId IS NOT NULL THEN
        SET affinityPU = this.sessionMap[request.sessionId]
        IF affinityPU IS NOT NULL AND affinityPU.IsHealthy() THEN
            RETURN affinityPU
        END IF
    END IF

    // Load-based routing: choose the least loaded PU
    SET units = this.GetHealthyProcessingUnits()
    SET selected = MinBy(units, FUNCTION(pu): RETURN pu.CurrentLoad() END FUNCTION)

    IF request.sessionId IS NOT NULL THEN
        SET this.sessionMap[request.sessionId] = selected
    END IF

    RETURN selected
END FUNCTION
```

## When to Use

- **High-volume, low-latency systems** where database bottlenecks are the
  primary scaling constraint (online bidding, ticketing, trading).
- **Systems with predictable data access patterns** where the working set
  fits in memory across processing units.
- **Applications requiring near-linear scaling** by adding processing units.
- **Session-heavy applications** (web sessions, shopping carts, game state)
  where session data must be available with minimal latency.
- **Systems where eventual database consistency** is acceptable as long as
  in-memory consistency is maintained.

## When NOT to Use

- **Data-heavy systems** where the dataset far exceeds available memory.
- **Strong consistency requirements** that demand immediate database
  durability for every write.
- **Simple applications** where a traditional database provides adequate
  performance without the complexity of data grids.
- **Small-scale systems** where the infrastructure cost of data grids and
  messaging grids is not justified.
- **Teams without distributed systems experience** who would struggle with
  the operational complexity.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Online auctions | Bidding platform | High concurrency, low latency, session state |
| Ticketing | Concert ticket sales | Massive concurrent demand during on-sale events |
| Gaming | Multiplayer game server | Session state, low latency, elastic scaling |
| Financial | High-frequency trading | Microsecond latency requirements |
| Social media | Live feed processing | High write throughput, eventually consistent reads |

## Trade-offs

### Advantages

- **Extreme scalability**: Near-linear scaling by adding processing units.
- **Low latency**: All reads/writes are in-memory.
- **No database bottleneck**: Database is used for durability, not performance.
- **Elastic**: Processing units can be added/removed dynamically.
- **Fault tolerance**: Data replicated across multiple units.

### Disadvantages

- **Complexity**: Data grids, replication, and messaging add significant
  operational burden.
- **Memory cost**: All active data must fit in memory across the grid.
- **Data consistency**: Replication introduces eventual consistency windows.
- **Testing difficulty**: Hard to simulate the full distributed environment.
- **Vendor dependency**: Data grid products are often specialized.

## Comparison with Related Patterns

| Dimension | Space-Based | Microservices | Monolith |
|-----------|------------|--------------|----------|
| Data location | In-memory (replicated) | Per-service DB | Shared DB |
| Scaling model | Add processing units | Scale per service | Scale entire app |
| Latency | Very low (memory) | Network hops | DB query time |
| Consistency | Eventual (grid) | Eventual (per service) | Strong (ACID) |
| Complexity | Very high | High | Low |

## Evolution and Variations

- **Partitioned Space**: Data is partitioned (sharded) across processing
  units rather than fully replicated. Reduces memory requirements.
- **Hybrid Model**: Hot data in the grid, cold data in the database.
  Data is loaded into the grid on demand and evicted based on usage.
- **Cloud-Native Space-Based**: Uses cloud-managed caching services and
  auto-scaling groups as the infrastructure layer.
- **Event-Sourced Space**: Combines space-based architecture with event
  sourcing for durable, replayable state management.

## Key Takeaways

1. Space-based architecture solves the database bottleneck problem by
   moving data into memory and distributing it across processing units.
2. The database becomes a backup mechanism, not the primary data store.
   This is a fundamental mindset shift from traditional architectures.
3. Data replication strategy (sync vs async, full vs partitioned) is the
   most critical design decision. It determines consistency, latency,
   and memory requirements.
4. This pattern is justified only for high-scale, low-latency systems.
   For moderate workloads, the complexity is not worth the benefit.
5. Operational maturity is a prerequisite. Teams must be comfortable with
   distributed systems, data grid management, and eventual consistency.
