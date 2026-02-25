# Event-Driven Architecture

## Overview and Intent

Event-Driven Architecture (EDA) structures a system around the production,
detection, consumption, and reaction to events. An event is a significant change
in state -- "order placed," "payment received," "temperature exceeded threshold."
Components communicate by emitting and subscribing to events rather than calling
each other directly.

The intent is to achieve loose coupling between components. Producers do not know
or care which consumers will react to their events, and consumers can be added or
removed without modifying producers. This decoupling enables systems that are
highly extensible, resilient to partial failures, and naturally suited to
asynchronous processing.

EDA is particularly powerful for systems where actions trigger cascading
reactions, where workload is unpredictable and bursty, or where real-time
responsiveness to state changes is critical. It is the foundation of modern
streaming platforms, IoT systems, and reactive applications.

## Structure and Components

### Component Diagram

```
+------------+     +------------+     +------------+
| Producer A |     | Producer B |     | Producer C |
| (Orders)   |     | (Payments) |     | (Sensors)  |
+-----+------+     +-----+------+     +-----+------+
      |                   |                   |
      | emit              | emit              | emit
      v                   v                   v
+-----+-------------------+-------------------+------+
|                    EVENT CHANNEL                    |
|   (Bus / Broker / Stream)                          |
|                                                     |
|   Topics:  [orders]  [payments]  [sensors]          |
+---+----------+----------+----------+----------+----+
    |          |          |          |          |
    v          v          v          v          v
+-------+ +-------+ +-------+ +-------+ +-------+
|Cons. 1| |Cons. 2| |Cons. 3| |Cons. 4| |Cons. 5|
|Notify | |Invent.| |Analyt.| |Billing| |Audit  |
+-------+ +-------+ +-------+ +-------+ +-------+
```

### Key Components

1. **Event Producer**: Detects a state change and publishes an event to
   the event channel. Producers are unaware of consumers.

2. **Event Channel**: The transport infrastructure (message broker, event
   bus, streaming platform) that routes events from producers to consumers.

3. **Event Consumer**: Subscribes to event types and reacts when matching
   events arrive. A single event may trigger multiple consumers.

4. **Event Store** (optional): Persistent log of all events, enabling
   replay, auditing, and temporal queries.

5. **Event Router / Processor**: Intermediate component that filters,
   transforms, enriches, or aggregates events before delivery.

## How It Works

```
  State change occurs
       |
       v
  +---------+       +------------------+
  |Producer |------>| Event Channel    |
  |emits    |       | (Topic: orders)  |
  |event    |       +--+----+----+-----+
  +---------+          |    |    |
                       |    |    |
          +------------+    |    +------------+
          |                 |                 |
          v                 v                 v
   +------+-----+   +------+-----+   +------+-----+
   | Consumer A  |   | Consumer B  |   | Consumer C  |
   | (Email)     |   | (Inventory) |   | (Analytics) |
   |             |   |             |   |             |
   | Send order  |   | Reserve     |   | Update      |
   | confirm.    |   | stock       |   | dashboard   |
   +-------------+   +-------------+   +-------------+
```

Step-by-step:

1. A state change occurs in the system (e.g., a new order is created).
2. The component responsible for that state change creates an event object
   containing the relevant data and metadata (timestamp, correlation ID).
3. The event is published to a named topic on the event channel.
4. The event channel delivers the event to all consumers subscribed to
   that topic.
5. Each consumer processes the event independently and asynchronously.
6. If a consumer fails, the event channel can retry delivery or route
   the event to a dead-letter queue for later processing.
7. Consumers may themselves produce new events, creating event chains.

## Pseudocode Example

```
// --- Event Definitions ---

STRUCTURE OrderPlacedEvent:
    eventId: String
    timestamp: DateTime
    correlationId: String
    orderId: String
    customerId: String
    items: List of OrderItem
    totalAmount: Decimal
END STRUCTURE

STRUCTURE PaymentProcessedEvent:
    eventId: String
    timestamp: DateTime
    correlationId: String
    orderId: String
    paymentId: String
    amount: Decimal
    status: String    // "SUCCESS" or "FAILED"
END STRUCTURE

// --- Event Producer ---

FUNCTION OrderService.PlaceOrder(orderData: OrderRequest) -> OrderResponse:
    // Business logic
    SET order = NEW Order
    SET order.id = GenerateUUID()
    SET order.customerId = orderData.customerId
    SET order.items = orderData.items
    SET order.status = "PLACED"
    SET order.total = CalculateTotal(orderData.items)

    OrderDatabase.Save(order)

    // Publish event -- producer does not know who will consume this
    SET event = NEW OrderPlacedEvent
    SET event.eventId = GenerateUUID()
    SET event.timestamp = Now()
    SET event.correlationId = CurrentContext().traceId
    SET event.orderId = order.id
    SET event.customerId = order.customerId
    SET event.items = order.items
    SET event.totalAmount = order.total

    EventChannel.Publish("orders.placed", event)

    RETURN OrderResponse(order.id, "PLACED")
END FUNCTION

// --- Event Channel Configuration ---

FUNCTION ConfigureEventChannel():
    SET channel = NEW EventChannel

    // Topic with consumer groups for parallel processing
    channel.CreateTopic("orders.placed", partitions: 8, retention: "7d")
    channel.CreateTopic("payments.processed", partitions: 4, retention: "30d")
    channel.CreateTopic("notifications.send", partitions: 4, retention: "1d")

    // Dead letter queue for failed events
    channel.CreateTopic("dead-letter", partitions: 1, retention: "90d")

    RETURN channel
END FUNCTION

// --- Event Consumers ---

FUNCTION InventoryConsumer.OnOrderPlaced(event: OrderPlacedEvent):
    LOG "Processing inventory for order " + event.orderId

    FOR EACH item IN event.items DO
        SET stock = InventoryDatabase.GetStock(item.productId)
        IF stock.available >= item.quantity THEN
            InventoryDatabase.Reserve(item.productId, item.quantity, event.orderId)
        ELSE
            // Publish compensation event
            EventChannel.Publish("inventory.insufficient", {
                orderId: event.orderId,
                productId: item.productId,
                requested: item.quantity,
                available: stock.available
            })
            RETURN
        END IF
    END FOR

    EventChannel.Publish("inventory.reserved", {
        orderId: event.orderId,
        correlationId: event.correlationId
    })
END FUNCTION

FUNCTION PaymentConsumer.OnOrderPlaced(event: OrderPlacedEvent):
    LOG "Processing payment for order " + event.orderId

    SET paymentResult = PaymentGateway.Charge(
        customerId: event.customerId,
        amount: event.totalAmount,
        reference: event.orderId
    )

    SET paymentEvent = NEW PaymentProcessedEvent
    SET paymentEvent.eventId = GenerateUUID()
    SET paymentEvent.timestamp = Now()
    SET paymentEvent.correlationId = event.correlationId
    SET paymentEvent.orderId = event.orderId
    SET paymentEvent.paymentId = paymentResult.transactionId
    SET paymentEvent.amount = event.totalAmount
    SET paymentEvent.status = paymentResult.status

    EventChannel.Publish("payments.processed", paymentEvent)
END FUNCTION

FUNCTION NotificationConsumer.OnOrderPlaced(event: OrderPlacedEvent):
    SET customer = CustomerDatabase.FindById(event.customerId)
    SET template = TemplateEngine.Load("order-confirmation")

    SET message = template.Render({
        customerName: customer.name,
        orderId: event.orderId,
        items: event.items,
        total: FormatCurrency(event.totalAmount)
    })

    NotificationService.SendEmail(customer.email, "Order Confirmation", message)
END FUNCTION

// --- Event Channel with Retry and Dead Letter ---

FUNCTION EventChannel.DeliverWithRetry(topic: String, event: Event, consumer: Consumer):
    SET maxRetries = 3
    SET attempt = 0

    LOOP
        SET attempt = attempt + 1
        TRY
            consumer.Handle(event)
            RETURN  // Success
        CATCH error
            LOG "Consumer " + consumer.name + " failed on attempt " + attempt
            IF attempt >= maxRetries THEN
                // Move to dead letter queue
                SET deadLetter = {
                    originalTopic: topic,
                    originalEvent: event,
                    consumer: consumer.name,
                    error: error.message,
                    failedAt: Now(),
                    attempts: attempt
                }
                this.Publish("dead-letter", deadLetter)
                RETURN
            END IF
            Sleep(ExponentialBackoff(attempt))
        END TRY
    END LOOP
END FUNCTION
```

## When to Use

- **Loosely coupled systems** where producers and consumers should evolve
  independently without coordination.
- **Real-time data processing** such as monitoring dashboards, alerting
  systems, and live analytics.
- **Systems with unpredictable or bursty workloads** where buffering events
  smooths demand spikes.
- **Complex workflows** where a single action triggers multiple independent
  reactions across the system.
- **Audit and compliance** where an immutable log of all state changes is
  required.

## When NOT to Use

- **Simple request-response workflows** where synchronous calls are clearer
  and easier to debug.
- **Systems requiring immediate consistency** where all effects of an
  action must be visible before responding to the client.
- **Small applications** where the overhead of an event channel infrastructure
  is not justified.
- **Teams without experience in asynchronous systems** where debugging
  event flows and handling eventual consistency is unfamiliar.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| IoT | Sensor data collection | High-volume, bursty, many-to-many data flow |
| E-commerce | Order fulfillment pipeline | Multiple independent steps triggered by order placement |
| Finance | Fraud detection | Real-time analysis of transaction events |
| Social media | Activity feeds | Fan-out of user actions to followers |
| Logistics | Package tracking | State change events propagate to multiple consumers |

## Trade-offs

### Advantages

- **Loose coupling**: Producers and consumers are independent.
- **Scalability**: Consumers can be scaled independently per topic.
- **Extensibility**: New consumers can be added without modifying producers.
- **Resilience**: Failed consumers can retry without affecting the rest.
- **Temporal decoupling**: Producers and consumers need not be online
  simultaneously.

### Disadvantages

- **Complexity**: Event flows are harder to trace than synchronous calls.
- **Eventual consistency**: State across consumers is not immediately
  consistent.
- **Debugging difficulty**: Distributed event chains are hard to debug.
- **Event ordering**: Guaranteeing event order across partitions is complex.
- **Infrastructure dependency**: Requires a reliable event channel.

## Comparison with Related Patterns

| Dimension | Event-Driven | Microservices (sync) | CQRS |
|-----------|-------------|---------------------|------|
| Coupling | Very loose | Moderate (API contracts) | Moderate |
| Consistency | Eventual | Per-service | Eventual |
| Communication | Async events | Sync HTTP/RPC | Events between models |
| Debugging | Harder (traces) | Moderate | Harder (dual models) |
| Scalability | High (per consumer) | High (per service) | High (per model) |

## Evolution and Variations

- **Simple Event Notification**: Events carry minimal data; consumers
  query the source for details. Simpler but adds coupling.
- **Event-Carried State Transfer**: Events carry full state, so consumers
  do not need to call back. Reduces coupling but increases event size.
- **Event Sourcing**: Events are the primary data store, not just
  notifications. Enables full state reconstruction from the event log.
- **CQRS + Event Driven**: Separates read and write models connected by
  events. Read models are eventually consistent projections.
- **Complex Event Processing (CEP)**: Detects patterns across multiple
  event streams in real time (e.g., "three failed logins within 5 minutes").

## Key Takeaways

1. Events represent facts about what happened, not commands about what
   to do. Name events in past tense: "OrderPlaced," not "PlaceOrder."
2. Design events to be self-contained. Include enough data for consumers
   to act without calling back to the producer.
3. Plan for failure at every stage: event production, channel delivery,
   and consumer processing. Use dead-letter queues and idempotent consumers.
4. Event ordering guarantees have significant performance costs. Use
   partitioning by entity ID to get ordering where it matters.
5. Invest in observability: correlation IDs, distributed tracing, and
   event flow visualization are essential for debugging event-driven systems.
