# CQRS (Command Query Responsibility Segregation)

## Overview and Intent

CQRS separates the model for reading data (queries) from the model for writing
data (commands) into two distinct models. Instead of using a single data model
for both reads and writes, the system maintains a write model optimized for
enforcing business rules and processing state changes, and a separate read model
optimized for serving queries and building views.

The intent is to allow each model to be optimized independently for its specific
purpose. Write models need strong consistency, validation, and complex business
rules. Read models need fast queries, denormalized data, and flexible projections.
A single model that tries to serve both purposes is forced into compromises that
satisfy neither optimally.

CQRS is not a top-level architecture; it is a pattern applied to a specific
bounded context or component where the read and write demands are sufficiently
different to justify the complexity of maintaining two models. It is often
combined with Event Sourcing, where domain events produced by the write side
are used to build and update the read side's projections.

## Structure and Components

### Component Diagram

```
                      +-------------------+
                      |   Client / UI     |
                      +---+----------+----+
                          |          |
                  Commands|          |Queries
                          v          v
                  +-------+--+  +---+--------+
                  | Command  |  | Query      |
                  | Handler  |  | Handler    |
                  +-------+--+  +---+--------+
                          |          |
                          v          v
               +----------+--+  +---+----------+
               | WRITE MODEL |  | READ MODEL   |
               |             |  |              |
               | - Aggregates|  | - Views      |
               | - Business  |  | - Projections|
               |   rules     |  | - Denorm.    |
               | - Validation|  |   data       |
               +------+------+  +------+-------+
                      |                ^
                      | Domain Events  | Projections
                      |                | build read model
                      +------->--------+
                        Event Bus / Store
```

### Key Components

1. **Command**: A request to change state. Commands are imperative ("PlaceOrder",
   "CancelShipment"). They carry the intent and data needed for the change.

2. **Command Handler**: Processes commands by loading aggregates, applying
   business rules, and persisting state changes. Produces domain events.

3. **Query**: A request to read data. Queries are declarative ("GetOrderById",
   "ListPendingOrders"). They never modify state.

4. **Query Handler**: Reads from the optimized read model and returns results.
   No business logic, no validation -- pure data retrieval.

5. **Write Model**: The domain model optimized for writes. Contains aggregates,
   entities, value objects, and business rules.

6. **Read Model (Projections)**: Denormalized views optimized for specific query
   patterns. Built and updated by processing domain events from the write side.

7. **Projection Builder**: Listens to domain events and updates the read model.
   Responsible for keeping the read side eventually consistent with the write side.

## How It Works

```
  WRITE SIDE:                           READ SIDE:

  Command: "Place Order"                Query: "Get Order Details"
       |                                      |
       v                                      v
  +----+------+                         +-----+-----+
  | Command   |                         | Query     |
  | Handler   |                         | Handler   |
  +----+------+                         +-----+-----+
       |                                      |
       v                                      v
  +----+------+                         +-----+-----+
  | Aggregate |                         | Read View |
  | (Order)   |                         | (Order    |
  +----+------+                         |  Detail)  |
       |                                +-----+-----+
       | events                               ^
       v                                      |
  +----+------+                         +-----+-----+
  | Write DB  |----> Event -------->    | Projection|
  | (Events)  |      Bus               | Builder   |
  +----------+                          +-----+-----+
                                              |
                                        +-----+-----+
                                        | Read DB   |
                                        | (Views)   |
                                        +-----------+
```

Step-by-step:

1. A command arrives (e.g., "Place Order with items A, B, C").
2. The command handler loads the relevant aggregate from the write store.
3. The aggregate validates the command and applies business rules.
4. If valid, the aggregate produces domain events ("OrderPlaced").
5. Events are persisted to the write store.
6. Events are published to the event bus.
7. Projection builders subscribe to events and update read models.
8. When a query arrives, the query handler reads directly from the
   denormalized read model -- fast, with no business logic overhead.

## Pseudocode Example

```
// === COMMANDS ===

STRUCTURE PlaceOrderCommand:
    orderId: String
    customerId: String
    items: List of OrderItemData
    shippingAddress: Address
END STRUCTURE

STRUCTURE CancelOrderCommand:
    orderId: String
    reason: String
END STRUCTURE

// === COMMAND HANDLERS ===

FUNCTION PlaceOrderCommandHandler.Handle(command: PlaceOrderCommand):
    // Validate command
    IF command.orderId IS NULL THEN
        RAISE ValidationError("Order ID is required")
    END IF
    IF Length(command.items) = 0 THEN
        RAISE ValidationError("Order must have at least one item")
    END IF

    // Load or create aggregate
    SET order = Order.Create(command.orderId, command.customerId)

    // Apply business rules through the aggregate
    FOR EACH item IN command.items DO
        order.AddItem(item.productId, item.name, item.quantity, item.price)
    END FOR
    order.SetShippingAddress(command.shippingAddress)
    order.Submit()

    // Persist (events or state)
    WriteStore.Save(order)

    // Publish domain events
    FOR EACH event IN order.PendingEvents() DO
        EventBus.Publish(event)
    END FOR
END FUNCTION

FUNCTION CancelOrderCommandHandler.Handle(command: CancelOrderCommand):
    SET order = WriteStore.Load(command.orderId)
    IF order IS NULL THEN
        RAISE NotFoundError("Order not found")
    END IF

    order.Cancel(command.reason)
    WriteStore.Save(order)

    FOR EACH event IN order.PendingEvents() DO
        EventBus.Publish(event)
    END FOR
END FUNCTION

// === QUERIES ===

STRUCTURE GetOrderDetailsQuery:
    orderId: String
END STRUCTURE

STRUCTURE ListOrdersByCustomerQuery:
    customerId: String
    status: String        // Optional filter
    page: Integer
    pageSize: Integer
END STRUCTURE

// === QUERY HANDLERS ===

FUNCTION GetOrderDetailsQueryHandler.Handle(query: GetOrderDetailsQuery) -> OrderDetailsView:
    // Read directly from the denormalized read model
    SET view = ReadStore.Get("order_details", query.orderId)
    IF view IS NULL THEN
        RAISE NotFoundError("Order not found")
    END IF
    RETURN view
END FUNCTION

FUNCTION ListOrdersByCustomerQueryHandler.Handle(query: ListOrdersByCustomerQuery) -> PagedResult:
    SET filters = {customerId: query.customerId}
    IF query.status IS NOT NULL THEN
        SET filters["status"] = query.status
    END IF

    SET results = ReadStore.Query("customer_orders",
        filters: filters,
        orderBy: "createdAt DESC",
        offset: (query.page - 1) * query.pageSize,
        limit: query.pageSize
    )

    SET total = ReadStore.Count("customer_orders", filters)

    RETURN PagedResult(
        items: results,
        page: query.page,
        pageSize: query.pageSize,
        totalCount: total
    )
END FUNCTION

// === READ MODEL (Projections) ===

STRUCTURE OrderDetailsView:
    orderId: String
    customerName: String
    customerEmail: String
    items: List of ItemView
    subtotal: Decimal
    tax: Decimal
    total: Decimal
    status: String
    statusHistory: List of StatusChange
    shippingAddress: String
    createdAt: DateTime
    lastUpdatedAt: DateTime
END STRUCTURE

STRUCTURE CustomerOrdersView:
    orderId: String
    customerId: String
    itemCount: Integer
    total: Decimal
    status: String
    createdAt: DateTime
END STRUCTURE

// === PROJECTION BUILDERS ===

FUNCTION OrderDetailsProjection.OnOrderPlaced(event: OrderPlacedEvent):
    SET view = NEW OrderDetailsView
    SET view.orderId = event.orderId
    SET view.customerName = CustomerReadStore.GetName(event.customerId)
    SET view.customerEmail = CustomerReadStore.GetEmail(event.customerId)
    SET view.items = EMPTY LIST
    SET view.status = "PLACED"
    SET view.statusHistory = [{status: "PLACED", at: event.timestamp}]
    SET view.createdAt = event.timestamp
    SET view.lastUpdatedAt = event.timestamp

    ReadStore.Put("order_details", event.orderId, view)
END FUNCTION

FUNCTION OrderDetailsProjection.OnItemAdded(event: OrderItemAddedEvent):
    SET view = ReadStore.Get("order_details", event.orderId)
    Append(view.items, {
        productId: event.productId,
        name: event.productName,
        quantity: event.quantity,
        unitPrice: event.unitPrice,
        lineTotal: event.quantity * event.unitPrice
    })
    SET view.subtotal = Sum(view.items, FUNCTION(i): RETURN i.lineTotal END FUNCTION)
    SET view.tax = view.subtotal * 0.08
    SET view.total = view.subtotal + view.tax
    SET view.lastUpdatedAt = event.timestamp

    ReadStore.Put("order_details", event.orderId, view)
END FUNCTION

FUNCTION OrderDetailsProjection.OnOrderCancelled(event: OrderCancelledEvent):
    SET view = ReadStore.Get("order_details", event.orderId)
    SET view.status = "CANCELLED"
    Append(view.statusHistory, {status: "CANCELLED", at: event.timestamp, reason: event.reason})
    SET view.lastUpdatedAt = event.timestamp

    ReadStore.Put("order_details", event.orderId, view)
END FUNCTION

FUNCTION CustomerOrdersProjection.OnOrderPlaced(event: OrderPlacedEvent):
    SET view = NEW CustomerOrdersView
    SET view.orderId = event.orderId
    SET view.customerId = event.customerId
    SET view.itemCount = event.itemCount
    SET view.total = event.total
    SET view.status = "PLACED"
    SET view.createdAt = event.timestamp

    ReadStore.Put("customer_orders", event.orderId, view)
END FUNCTION

FUNCTION CustomerOrdersProjection.OnOrderCancelled(event: OrderCancelledEvent):
    SET view = ReadStore.Get("customer_orders", event.orderId)
    SET view.status = "CANCELLED"
    ReadStore.Put("customer_orders", event.orderId, view)
END FUNCTION

// === APPLICATION WIRING ===

FUNCTION ConfigureCQRS():
    // Write side
    SET writeStore = NEW EventStore("write-db-connection")
    SET commandBus = NEW CommandBus
    commandBus.Register("PlaceOrder", NEW PlaceOrderCommandHandler(writeStore))
    commandBus.Register("CancelOrder", NEW CancelOrderCommandHandler(writeStore))

    // Read side
    SET readStore = NEW ReadDatabase("read-db-connection")
    SET eventBus = NEW EventBus

    // Wire projections
    SET orderDetails = NEW OrderDetailsProjection(readStore)
    eventBus.Subscribe("OrderPlaced", orderDetails.OnOrderPlaced)
    eventBus.Subscribe("OrderItemAdded", orderDetails.OnItemAdded)
    eventBus.Subscribe("OrderCancelled", orderDetails.OnOrderCancelled)

    SET customerOrders = NEW CustomerOrdersProjection(readStore)
    eventBus.Subscribe("OrderPlaced", customerOrders.OnOrderPlaced)
    eventBus.Subscribe("OrderCancelled", customerOrders.OnOrderCancelled)

    // Query handlers
    SET queryBus = NEW QueryBus
    queryBus.Register("GetOrderDetails", NEW GetOrderDetailsQueryHandler(readStore))
    queryBus.Register("ListOrdersByCustomer", NEW ListOrdersByCustomerQueryHandler(readStore))

    RETURN {commandBus, queryBus, eventBus}
END FUNCTION
```

## When to Use

- **Read-heavy systems** where read patterns are vastly different from
  write patterns (e.g., complex reports over simple write operations).
- **Systems with different scaling needs** for reads versus writes (scale
  read replicas independently).
- **Complex domain logic on writes** with simple, denormalized views on
  reads. Each side can be optimized without compromising the other.
- **Systems where eventual consistency is acceptable** for the read side.
- **Multi-view requirements** where the same data must be presented in
  many different formats or aggregations.

## When NOT to Use

- **Simple CRUD applications** where reads and writes use the same model
  and the complexity of two models is not justified.
- **Systems requiring immediate consistency** between writes and reads
  (user writes data and must see it instantly reflected).
- **Small teams** where maintaining two models, projection builders, and
  an event bus doubles the development and operational effort.
- **Low-traffic systems** where the performance benefits of separate
  read/write models are negligible.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| E-commerce | Product catalog + order management | Complex write rules, highly optimized search/browse reads |
| Finance | Trading + reporting | Real-time writes, complex analytical queries |
| Social media | Post creation + news feed | Simple writes, complex fan-out read projections |
| Healthcare | Patient records + clinical reports | Regulated writes, diverse reporting views |
| Analytics | Data ingestion + dashboard queries | High-throughput writes, complex aggregation reads |

## Trade-offs

### Advantages

- **Optimized models**: Each side optimized for its specific workload.
- **Independent scaling**: Scale reads and writes separately.
- **Simplified queries**: Read models are pre-computed, denormalized views.
- **Flexibility**: Multiple read models for different query patterns.
- **Performance**: Write side does not carry query overhead and vice versa.

### Disadvantages

- **Eventual consistency**: Read model lags behind write model.
- **Complexity**: Two models, projection builders, event bus infrastructure.
- **Data duplication**: Same data stored in different formats.
- **Debugging difficulty**: Tracing data flow across models is complex.
- **Synchronization bugs**: Projection builder errors cause stale views.

## Comparison with Related Patterns

| Dimension | CQRS | Event Sourcing | Traditional CRUD |
|-----------|------|---------------|-----------------|
| Data model | Separate read/write | Event log + projections | Single model |
| Consistency | Eventual (read side) | Eventual (projections) | Immediate |
| State storage | Current state (write) + views (read) | Events only | Current state |
| Query performance | Optimized projections | Must build projections | Direct queries |
| Complexity | High | Very high | Low |

## Evolution and Variations

- **Simple CQRS**: Separate read and write models against the same database.
  The simplest form, using database views or separate tables.
- **CQRS + Event Sourcing**: Write side stores events; read side builds
  projections from the event stream. The most powerful but complex variant.
- **CQRS + Messaging**: Asynchronous event propagation via message broker.
  Provides temporal decoupling between write and read processing.
- **Polyglot CQRS**: Different storage technologies for read and write
  (e.g., relational for writes, search index for reads).

## Key Takeaways

1. CQRS is justified when read and write patterns are fundamentally
   different. If they are similar, a single model is simpler and better.
2. Eventual consistency is the core trade-off. The read model will always
   lag behind the write model by some amount. Design the UI to communicate
   this clearly to users.
3. Projection builders must be idempotent. Events may be delivered more
   than once; replaying events should produce the same read model state.
4. Start without CQRS and introduce it only when you observe pain: slow
   queries, conflicting optimization needs, or scaling bottlenecks.
5. CQRS pairs naturally with Event Sourcing and Domain-Driven Design.
   Together, they form a powerful toolkit for complex domain systems,
   but the combined complexity should not be underestimated.
