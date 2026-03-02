# Event Sourcing Architecture

## Overview and Intent

Event Sourcing is a pattern where the state of an application is determined
by a sequence of events rather than by storing only the current state. Instead
of updating a record in place (overwriting previous values), every state change
is captured as an immutable event appended to an event store. The current state
of any entity can be reconstructed at any time by replaying its events from the
beginning.

The intent is to provide a complete, auditable, and replayable history of
everything that happened in the system. Traditional state-based storage loses
history -- once a record is updated, the previous value is gone. Event sourcing
preserves every change, enabling temporal queries ("what was the account balance
on March 15th?"), audit trails, debugging, and the ability to rebuild the
entire system state from the event log.

Event sourcing is most valuable in domains where the history of changes is
as important as the current state: financial systems, healthcare records,
legal document management, and any system with regulatory audit requirements.
It pairs naturally with CQRS, where the event log feeds projections that
serve as optimized read models.

## Structure and Components

### Component Diagram

```
   Command
      |
      v
+-----+------+
| Aggregate   |  1. Load events from store
| (Domain     |  2. Replay to build current state
|  Object)    |  3. Validate command
+-----+-------+  4. Produce new events
      |
      | new events
      v
+-----+-------+
| EVENT STORE |  5. Append events (immutable log)
|             |
| [Event 1]  |  Events are NEVER modified or deleted.
| [Event 2]  |  The store is append-only.
| [Event 3]  |
| [  ...   ]  |
+------+------+
       |
       | published to subscribers
       |
  +----+----+--------+--------+
  |         |        |        |
  v         v        v        v
+----+  +----+  +----+  +------+
|Proj|  |Proj|  |Proj|  |Notif.|
|  A |  |  B |  |  C |  |Svc   |
+----+  +----+  +----+  +------+
  |       |       |
  v       v       v
[Read DB A]  [Read DB B]  [Read DB C]
```

### Key Components

1. **Event**: An immutable record of something that happened. Contains the
   event type, aggregate ID, data payload, timestamp, and sequence number.

2. **Event Store**: An append-only persistent log of events. Supports loading
   events by aggregate ID and appending with optimistic concurrency.

3. **Aggregate**: A domain object that loads its state by replaying past
   events and produces new events in response to commands.

4. **Snapshot**: A periodic capture of aggregate state to avoid replaying
   the entire event history on every load.

5. **Projection**: A read-side component that subscribes to events and
   builds denormalized views for specific query patterns.

## How It Works

```
  Command: "Deposit 500 into account A"
       |
       v
  +----+------+
  | Load      |  1. Read all events for account A from event store
  | Aggregate |     [AccountOpened(balance: 0)]
  +----+------+     [FundsDeposited(amount: 1000)]
       |            [FundsWithdrawn(amount: 200)]
       v
  +----+------+  2. Replay events to build current state:
  | Replay    |     AccountOpened  -> balance = 0
  | Events    |     FundsDeposited -> balance = 1000
  +----+------+     FundsWithdrawn -> balance = 800
       |
       v            Current state: balance = 800
  +----+------+
  | Execute   |  3. Validate: balance + 500 = 1300 (valid)
  | Command   |  4. Produce new event: FundsDeposited(amount: 500)
  +----+------+
       |
       v
  +----+------+
  | Event     |  5. Append: [FundsDeposited(amount: 500)]
  | Store     |     with expected version = 3 (optimistic concurrency)
  +----+------+
       |
       v
  | Publish   |  6. Notify projections and subscribers
```

## Pseudocode Example

```
// === EVENT DEFINITIONS ===

STRUCTURE DomainEvent:
    eventId: String
    aggregateId: String
    eventType: String
    data: Map
    timestamp: DateTime
    version: Integer
END STRUCTURE

// === EVENT STORE ===

INTERFACE EventStore:
    FUNCTION LoadEvents(aggregateId: String) -> List of DomainEvent
    FUNCTION LoadEventsSince(aggregateId: String, version: Integer) -> List of DomainEvent
    FUNCTION AppendEvents(aggregateId: String, events: List of DomainEvent, expectedVersion: Integer) -> Boolean
END INTERFACE

FUNCTION EventStoreImpl.AppendEvents(aggregateId: String, events: List of DomainEvent, expectedVersion: Integer) -> Boolean:
    // Optimistic concurrency check
    SET currentVersion = this.GetCurrentVersion(aggregateId)
    IF currentVersion != expectedVersion THEN
        RAISE ConcurrencyConflict(
            "Expected version " + expectedVersion + " but found " + currentVersion
        )
    END IF

    SET nextVersion = expectedVersion
    FOR EACH event IN events DO
        SET nextVersion = nextVersion + 1
        SET event.version = nextVersion
        SET event.eventId = GenerateUUID()
        SET event.timestamp = Now()
        this.persistence.Append(aggregateId, event)
    END FOR

    FOR EACH event IN events DO
        this.eventBus.Publish(event)
    END FOR
    RETURN TRUE
END FUNCTION

// === AGGREGATE (Event-Sourced) ===

STRUCTURE Account:
    id: String
    ownerId: String
    balance: Decimal
    currency: String
    status: String
    version: Integer
    pendingEvents: List of DomainEvent
END STRUCTURE

FUNCTION Account.LoadFromHistory(events: List of DomainEvent) -> Account:
    SET account = NEW Account
    SET account.pendingEvents = EMPTY LIST
    SET account.version = 0
    FOR EACH event IN events DO
        account.Apply(event)
        SET account.version = event.version
    END FOR
    RETURN account
END FUNCTION

FUNCTION Account.Apply(event: DomainEvent):
    IF event.eventType = "AccountOpened" THEN
        SET this.id = event.aggregateId
        SET this.ownerId = event.data.ownerId
        SET this.balance = event.data.initialBalance
        SET this.currency = event.data.currency
        SET this.status = "ACTIVE"
    ELSE IF event.eventType = "FundsDeposited" THEN
        SET this.balance = this.balance + event.data.amount
    ELSE IF event.eventType = "FundsWithdrawn" THEN
        SET this.balance = this.balance - event.data.amount
    ELSE IF event.eventType = "AccountFrozen" THEN
        SET this.status = "FROZEN"
    END IF
END FUNCTION

FUNCTION Account.Deposit(amount: Decimal, reference: String):
    IF this.status != "ACTIVE" THEN
        RAISE DomainError("Cannot deposit to " + this.status + " account")
    END IF
    IF amount <= 0 THEN
        RAISE DomainError("Deposit amount must be positive")
    END IF

    SET event = NEW DomainEvent
    SET event.aggregateId = this.id
    SET event.eventType = "FundsDeposited"
    SET event.data = {amount: amount, reference: reference}

    this.Apply(event)
    Append(this.pendingEvents, event)
END FUNCTION

FUNCTION Account.Withdraw(amount: Decimal, reference: String):
    IF this.status != "ACTIVE" THEN
        RAISE DomainError("Cannot withdraw from " + this.status + " account")
    END IF
    IF amount <= 0 THEN
        RAISE DomainError("Withdrawal amount must be positive")
    END IF
    IF amount > this.balance THEN
        RAISE DomainError("Insufficient funds")
    END IF

    SET event = NEW DomainEvent
    SET event.aggregateId = this.id
    SET event.eventType = "FundsWithdrawn"
    SET event.data = {amount: amount, reference: reference}

    this.Apply(event)
    Append(this.pendingEvents, event)
END FUNCTION

// === COMMAND HANDLER ===

FUNCTION DepositHandler.Handle(command: DepositCommand):
    SET events = eventStore.LoadEvents(command.accountId)
    IF Length(events) = 0 THEN
        RAISE NotFoundError("Account not found")
    END IF

    SET account = Account.LoadFromHistory(events)
    account.Deposit(command.amount, command.reference)

    eventStore.AppendEvents(account.id, account.pendingEvents, expectedVersion: account.version)
END FUNCTION

// === SNAPSHOT OPTIMIZATION ===

STRUCTURE Snapshot:
    aggregateId: String
    version: Integer
    state: Account
    createdAt: DateTime
END STRUCTURE

FUNCTION LoadAccountWithSnapshot(accountId: String) -> Account:
    SET snapshot = SnapshotStore.GetLatest(accountId)
    IF snapshot IS NOT NULL THEN
        SET account = snapshot.state
        SET newEvents = eventStore.LoadEventsSince(accountId, snapshot.version)
        FOR EACH event IN newEvents DO
            account.Apply(event)
            SET account.version = event.version
        END FOR
        RETURN account
    ELSE
        SET events = eventStore.LoadEvents(accountId)
        RETURN Account.LoadFromHistory(events)
    END IF
END FUNCTION

// === PROJECTION (Read Model Builder) ===

FUNCTION AccountBalanceProjection.Handle(event: DomainEvent):
    IF event.eventType = "AccountOpened" THEN
        ReadDB.Insert("account_balances", {
            accountId: event.aggregateId,
            ownerId: event.data.ownerId,
            balance: event.data.initialBalance,
            currency: event.data.currency,
            status: "ACTIVE",
            lastUpdated: event.timestamp
        })
    ELSE IF event.eventType = "FundsDeposited" THEN
        SET current = ReadDB.GetField("account_balances", event.aggregateId, "balance")
        ReadDB.Update("account_balances",
            where: {accountId: event.aggregateId},
            set: {balance: current + event.data.amount, lastUpdated: event.timestamp}
        )
    ELSE IF event.eventType = "FundsWithdrawn" THEN
        SET current = ReadDB.GetField("account_balances", event.aggregateId, "balance")
        ReadDB.Update("account_balances",
            where: {accountId: event.aggregateId},
            set: {balance: current - event.data.amount, lastUpdated: event.timestamp}
        )
    ELSE IF event.eventType = "AccountFrozen" THEN
        ReadDB.Update("account_balances",
            where: {accountId: event.aggregateId},
            set: {status: "FROZEN", lastUpdated: event.timestamp}
        )
    END IF
END FUNCTION

// === TEMPORAL QUERY (unique to event sourcing) ===

FUNCTION GetAccountBalanceAtDate(accountId: String, targetDate: DateTime) -> Decimal:
    SET events = eventStore.LoadEvents(accountId)
    SET account = NEW Account
    FOR EACH event IN events DO
        IF event.timestamp > targetDate THEN
            BREAK
        END IF
        account.Apply(event)
    END FOR
    RETURN account.balance
END FUNCTION

FUNCTION GetAccountHistory(accountId: String) -> List of HistoryEntry:
    SET events = eventStore.LoadEvents(accountId)
    SET history = EMPTY LIST
    SET runningBalance = 0
    FOR EACH event IN events DO
        IF event.eventType = "AccountOpened" THEN
            SET runningBalance = event.data.initialBalance
        ELSE IF event.eventType = "FundsDeposited" THEN
            SET runningBalance = runningBalance + event.data.amount
        ELSE IF event.eventType = "FundsWithdrawn" THEN
            SET runningBalance = runningBalance - event.data.amount
        END IF
        Append(history, {
            timestamp: event.timestamp,
            eventType: event.eventType,
            data: event.data,
            balanceAfter: runningBalance
        })
    END FOR
    RETURN history
END FUNCTION
```

## When to Use

- **Audit-critical domains** (finance, healthcare, legal) where a complete
  history of every change is required.
- **Temporal queries** where you need to answer "what was the state at
  time T?" or "how did we get to this state?"
- **Complex event processing** where downstream systems need to react to
  detailed state change events.
- **Debugging and replay** where the ability to reproduce any past state
  is valuable for diagnosing issues.
- **Systems with CQRS** where events naturally connect the write model
  to read model projections.

## When NOT to Use

- **Simple CRUD systems** where current state is the only concern and
  history has no business value.
- **High-frequency update systems** where the event log would grow
  impractically fast without extreme optimization.
- **Systems requiring immediate consistency** where the projection lag
  is unacceptable.
- **Teams without event modeling experience** where the paradigm shift
  from state-based thinking is too steep.
- **Domains with delete requirements** (such as privacy regulations
  requiring data erasure) that conflict with the append-only model.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Banking | Account transaction ledger | Complete audit trail, regulatory compliance |
| Healthcare | Patient medical record | Full history of diagnoses, treatments |
| E-commerce | Shopping cart with undo | Replay actions, analyze abandonment |
| Version control | Source code history | Every change recorded, any version reconstructable |
| Supply chain | Package tracking history | Complete journey log, temporal queries |

## Trade-offs

### Advantages

- **Complete audit trail**: Every change is recorded permanently.
- **Temporal queries**: Reconstruct state at any point in time.
- **Debugging**: Replay events to reproduce any bug.
- **Event-driven integration**: Events naturally available for consumers.
- **No data loss**: State is never overwritten, only appended.

### Disadvantages

- **Storage growth**: Event store grows indefinitely.
- **Replay performance**: Long event histories slow aggregate loading.
- **Schema evolution**: Changing event formats requires migration strategies.
- **Complexity**: Fundamentally different paradigm from traditional CRUD.
- **Eventual consistency**: Projections lag behind the event store.
- **Delete challenges**: Privacy regulations conflict with immutability.

## Comparison with Related Patterns

| Dimension | Event Sourcing | Traditional CRUD | CQRS (without ES) |
|-----------|---------------|-----------------|-------------------|
| State storage | Events (append-only) | Current state (mutable) | Current state (R/W split) |
| History | Complete | Lost on update | Partial (if logged) |
| Temporal queries | Native | Not possible | Not possible |
| Storage efficiency | Lower (all events) | Higher (current only) | Medium |
| Complexity | Very high | Low | High |

## Evolution and Variations

- **Event Sourcing + CQRS**: The most common combination. Events from the
  write side feed projections that serve as optimized read models.
- **Event Sourcing + Snapshots**: Periodic snapshots reduce replay time.
- **Event Upcasting**: Transforming old event formats to new formats during
  replay, supporting schema evolution without data migration.
- **Crypto-Deletion**: Encrypting event data with per-entity keys and
  destroying the key when deletion is required.
- **Event-Sourced Saga**: Long-running processes coordinated across
  aggregates using events as their state transitions.

## Key Takeaways

1. Events are facts, not commands. They describe what happened, not what
   should happen. They are immutable and append-only.
2. The event store is the source of truth. All other representations
   (projections, caches) are derived from it.
3. Aggregate replay is the fundamental mechanism. Every state is built by
   replaying events, guaranteeing consistency and enabling temporal queries.
4. Snapshots are an optimization, not a replacement for the event log.
5. Schema evolution is the hardest operational challenge. Plan for event
   versioning and upcasting from the start, because event formats will
   change as the domain model evolves.
