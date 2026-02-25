# Domain-Driven Design (DDD)

## Overview and Intent

Domain-Driven Design is an approach to software development that places the
business domain at the center of all architectural decisions. It provides a
collection of patterns for modeling complex business domains, structuring code
around those models, and managing the boundaries between different parts of a
large system.

The intent of DDD is to align the software model with the mental model of domain
experts. When developers and domain experts share a common language (the
"Ubiquitous Language") and the code reflects that language, the software becomes
easier to understand, discuss, and evolve. DDD is most valuable in systems with
complex business logic where getting the model right is critical to success.

DDD operates at two levels: strategic design (how to divide a large system into
bounded contexts) and tactical design (how to model entities, value objects,
aggregates, and domain events within a bounded context). Both levels work
together to create systems that are maintainable, scalable, and aligned with
business needs.

## Structure and Components

### Component Diagram

```
+====================================================================+
|                         SYSTEM BOUNDARY                             |
|                                                                     |
|  +-----------------------+       +-----------------------+          |
|  | BOUNDED CONTEXT:      |       | BOUNDED CONTEXT:      |          |
|  | Order Management      |       | Shipping               |          |
|  |                        |       |                        |          |
|  | +------------------+  |       | +------------------+  |          |
|  | | Aggregate:       |  |       | | Aggregate:       |  |          |
|  | | Order            |  |       | | Shipment         |  |          |
|  | | +------+         |  |       | | +------+         |  |          |
|  | | |Entity|  [Root] |  |       | | |Entity|  [Root] |  |          |
|  | | +------+         |  |       | | +------+         |  |          |
|  | | +------+ +-----+ |  |       | | +------+ +-----+ |  |          |
|  | | |Entity| | VO  | |  |       | | |Entity| | VO  | |  |          |
|  | | +------+ +-----+ |  |       | | +------+ +-----+ |  |          |
|  | +------------------+  |       | +------------------+  |          |
|  |                        |       |                        |          |
|  | +------------------+  |       | +------------------+  |          |
|  | | Domain Service   |  |       | | Domain Service   |  |          |
|  | +------------------+  |       | +------------------+  |          |
|  |                        |       |                        |          |
|  | +------------------+  |       | +------------------+  |          |
|  | | Repository       |  |       | | Repository       |  |          |
|  | +------------------+  |       | +------------------+  |          |
|  +-----------+------------+       +-----------+------------+          |
|              |                                |                      |
|              | Domain Events                  | Domain Events        |
|              +-------->  CONTEXT MAP  <-------+                      |
|                                                                     |
+====================================================================+
```

### Key Components

1. **Bounded Context**: A boundary within which a particular domain model
   is defined and applicable. Different contexts may model the same
   real-world concept differently.

2. **Aggregate**: A cluster of domain objects treated as a unit for data
   changes. One entity is the Aggregate Root, the only entry point for
   external access.

3. **Entity**: An object with a persistent identity that runs through
   time and across representations.

4. **Value Object**: An immutable object defined by its attributes rather
   than an identity. Two value objects with the same attributes are equal.

5. **Domain Service**: Stateless operations that do not naturally belong
   to any single entity or value object.

6. **Domain Event**: A record of something that happened in the domain.
   Used for communication between aggregates and bounded contexts.

7. **Repository**: An abstraction that provides collection-like access
   to aggregates, hiding persistence mechanics.

## How It Works

```
  Command: "Place Order"
       |
       v
  +---------+
  | Application   |  1. Load aggregate from repository
  | Service       |  2. Invoke domain operation
  +---------+     |  3. Save aggregate through repository
       |          |  4. Publish domain events
       +----------+
       |
       v
  +----+------+
  | Aggregate |  5. Validate invariants
  | Root      |  6. Apply business rules
  | (Order)   |  7. Produce domain events
  +----+------+
       |
       v
  +----+------+
  | Repository|  8. Persist entire aggregate atomically
  +----+------+
       |
       v
  +----+------+
  | Event     |  9. Publish events to other contexts
  | Dispatcher|
  +----+------+
       |
  +----+----+-------+
  |         |       |
  v         v       v
Context A  Context B  Context C
(reacts)   (reacts)   (reacts)
```

## Pseudocode Example

```
// === VALUE OBJECTS ===

STRUCTURE Money:
    amount: Decimal
    currency: String
END STRUCTURE

FUNCTION Money.Add(other: Money) -> Money:
    IF this.currency != other.currency THEN
        RAISE DomainError("Cannot add different currencies")
    END IF
    RETURN NEW Money(this.amount + other.amount, this.currency)
END FUNCTION

FUNCTION Money.Subtract(other: Money) -> Money:
    IF this.currency != other.currency THEN
        RAISE DomainError("Cannot subtract different currencies")
    END IF
    IF other.amount > this.amount THEN
        RAISE DomainError("Result would be negative")
    END IF
    RETURN NEW Money(this.amount - other.amount, this.currency)
END FUNCTION

FUNCTION Money.Equals(other: Money) -> Boolean:
    RETURN this.amount = other.amount AND this.currency = other.currency
END FUNCTION

STRUCTURE Address:
    street: String
    city: String
    state: String
    postalCode: String
    country: String
END STRUCTURE

// === ENTITY ===

STRUCTURE OrderItem:
    id: String
    productId: String
    productName: String
    quantity: Integer
    unitPrice: Money
END STRUCTURE

FUNCTION OrderItem.LineTotal() -> Money:
    RETURN NEW Money(this.unitPrice.amount * this.quantity, this.unitPrice.currency)
END FUNCTION

// === AGGREGATE ROOT ===

STRUCTURE Order:
    id: String                          // Identity
    customerId: String
    items: List of OrderItem
    shippingAddress: Address            // Value Object
    status: OrderStatus
    total: Money                        // Value Object
    domainEvents: List of DomainEvent   // Pending events
    createdAt: DateTime
    version: Integer                    // Optimistic concurrency
END STRUCTURE

FUNCTION Order.Create(customerId: String, address: Address) -> Order:
    SET order = NEW Order
    SET order.id = GenerateId()
    SET order.customerId = customerId
    SET order.items = EMPTY LIST
    SET order.shippingAddress = address
    SET order.status = DRAFT
    SET order.total = NEW Money(0, "USD")
    SET order.domainEvents = EMPTY LIST
    SET order.createdAt = Now()
    SET order.version = 0
    RETURN order
END FUNCTION

FUNCTION Order.AddItem(productId: String, name: String, qty: Integer, price: Money):
    // Invariant: cannot modify non-draft orders
    IF this.status != DRAFT THEN
        RAISE DomainError("Cannot add items to a " + this.status + " order")
    END IF

    // Invariant: quantity must be positive
    IF qty <= 0 THEN
        RAISE DomainError("Quantity must be positive")
    END IF

    // Invariant: max 50 items per order
    IF Length(this.items) >= 50 THEN
        RAISE DomainError("Order cannot have more than 50 items")
    END IF

    SET item = NEW OrderItem
    SET item.id = GenerateId()
    SET item.productId = productId
    SET item.productName = name
    SET item.quantity = qty
    SET item.unitPrice = price

    Append(this.items, item)
    this.RecalculateTotal()
END FUNCTION

FUNCTION Order.Submit():
    IF this.status != DRAFT THEN
        RAISE DomainError("Only draft orders can be submitted")
    END IF
    IF Length(this.items) = 0 THEN
        RAISE DomainError("Cannot submit an empty order")
    END IF

    SET this.status = SUBMITTED

    // Record domain event
    Append(this.domainEvents, NEW OrderSubmittedEvent(
        orderId: this.id,
        customerId: this.customerId,
        total: this.total,
        itemCount: Length(this.items),
        occurredAt: Now()
    ))
END FUNCTION

FUNCTION Order.Cancel(reason: String):
    IF this.status = SHIPPED OR this.status = DELIVERED THEN
        RAISE DomainError("Cannot cancel a shipped or delivered order")
    END IF

    SET this.status = CANCELLED

    Append(this.domainEvents, NEW OrderCancelledEvent(
        orderId: this.id,
        reason: reason,
        occurredAt: Now()
    ))
END FUNCTION

FUNCTION Order.RecalculateTotal():
    SET sum = NEW Money(0, "USD")
    FOR EACH item IN this.items DO
        SET sum = sum.Add(item.LineTotal())
    END FOR
    SET this.total = sum
END FUNCTION

// === DOMAIN SERVICE ===

FUNCTION PricingService.ApplyDiscount(order: Order, customer: Customer) -> Money:
    SET discount = NEW Money(0, order.total.currency)

    // Business rule: loyalty discount
    IF customer.loyaltyTier = "GOLD" THEN
        SET discount = NEW Money(order.total.amount * 0.10, order.total.currency)
    ELSE IF customer.loyaltyTier = "SILVER" THEN
        SET discount = NEW Money(order.total.amount * 0.05, order.total.currency)
    END IF

    // Business rule: bulk discount
    SET totalQuantity = 0
    FOR EACH item IN order.items DO
        SET totalQuantity = totalQuantity + item.quantity
    END FOR
    IF totalQuantity > 100 THEN
        SET bulkDiscount = NEW Money(order.total.amount * 0.03, order.total.currency)
        IF bulkDiscount.amount > discount.amount THEN
            SET discount = bulkDiscount
        END IF
    END IF

    RETURN discount
END FUNCTION

// === REPOSITORY ===

INTERFACE OrderRepository:
    FUNCTION FindById(orderId: String) -> Order
    FUNCTION Save(order: Order)
    FUNCTION NextIdentity() -> String
END INTERFACE

// === APPLICATION SERVICE ===

FUNCTION SubmitOrderHandler(command: SubmitOrderCommand):
    // Load aggregate
    SET order = orderRepository.FindById(command.orderId)
    IF order IS NULL THEN
        RAISE NotFoundError("Order " + command.orderId + " not found")
    END IF

    // Invoke domain operation
    order.Submit()

    // Persist aggregate (entire aggregate is saved atomically)
    orderRepository.Save(order)

    // Publish domain events
    FOR EACH event IN order.domainEvents DO
        eventDispatcher.Publish(event)
    END FOR
END FUNCTION

// === CONTEXT MAP: Anti-Corruption Layer ===

FUNCTION ShippingContextAdapter.OnOrderSubmitted(event: OrderSubmittedEvent):
    // Translate from Order context to Shipping context
    SET shipmentRequest = NEW ShipmentRequest
    SET shipmentRequest.referenceId = event.orderId
    SET shipmentRequest.destination = AddressTranslator.ToShippingAddress(
        orderRepository.FindById(event.orderId).shippingAddress
    )
    SET shipmentRequest.priority = DeterminePriority(event.total)

    ShippingService.CreateShipment(shipmentRequest)
END FUNCTION
```

## When to Use

- **Complex business domains** with rich rules, many edge cases, and
  domain experts who need to collaborate with developers.
- **Systems where the model is the competitive advantage** and getting
  it wrong has high business cost.
- **Large teams** that need to divide work along bounded context boundaries
  to reduce coordination overhead.
- **Long-lived systems** that must evolve as business rules change.
- **Multiple bounded contexts** with different models of the same concept.

## When NOT to Use

- **Simple CRUD applications** where the domain is thin and data mapping
  dominates the implementation.
- **Highly technical domains** (infrastructure tools, network protocols)
  where DDD's business modeling vocabulary does not apply.
- **Small projects** where the overhead of aggregates, value objects, and
  bounded contexts is not justified.
- **Teams without domain expert access** who cannot build a shared
  ubiquitous language.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Banking | Loan origination | Complex rules, multiple contexts (credit, compliance, servicing) |
| Healthcare | Hospital management | Patient, billing, scheduling are distinct bounded contexts |
| Insurance | Policy lifecycle | Rich domain with underwriting, claims, renewals |
| Supply chain | Warehouse management | Inventory, picking, shipping are distinct contexts |
| E-commerce | Large marketplace | Catalog, ordering, payments, fulfillment contexts |

## Trade-offs

### Advantages

- **Business alignment**: Code mirrors how domain experts think.
- **Encapsulated complexity**: Aggregates enforce invariants internally.
- **Scalable teams**: Bounded contexts allow independent team ownership.
- **Evolvability**: Well-modeled domains adapt to rule changes easily.
- **Ubiquitous language**: Reduces miscommunication between developers
  and domain experts.

### Disadvantages

- **Steep learning curve**: DDD concepts require significant investment.
- **Over-engineering risk**: Applying DDD to simple domains adds waste.
- **Aggregate design difficulty**: Getting aggregate boundaries right is
  hard and has lasting consequences.
- **Persistence complexity**: Mapping aggregates to storage is non-trivial.
- **Requires domain experts**: DDD is ineffective without genuine domain
  knowledge input.

## Comparison with Related Patterns

| Dimension | DDD | Clean Architecture | Microservices |
|-----------|-----|-------------------|---------------|
| Focus | Business model accuracy | Dependency isolation | Deployment independence |
| Boundary type | Bounded contexts | Concentric circles | Service boundaries |
| Team alignment | Domain teams | Layer/feature teams | Service teams |
| Data handling | Aggregate consistency | Gateway abstraction | Per-service databases |
| Key artifact | Domain model | Use case interactor | Service API |

## Evolution and Variations

- **Event-Sourced DDD**: Aggregates store domain events instead of current
  state. Enables audit trails and temporal queries.
- **CQRS + DDD**: Separates the write model (rich aggregates) from the
  read model (optimized query projections).
- **Lightweight DDD**: Uses tactical patterns (entities, value objects)
  without the full strategic framework. Common in smaller systems.
- **Functional DDD**: Models aggregates as pure functions that take
  commands and return events, avoiding mutable state.

## Key Takeaways

1. Ubiquitous Language is the foundation. If the code does not use the
   same terms as the domain experts, the model is wrong.
2. Bounded contexts are the most important strategic pattern. They
   prevent large models from becoming incoherent.
3. Aggregates define transactional boundaries. Keep them small and
   design them around invariants, not data relationships.
4. Value objects should be the default. Use entities only when identity
   matters. Most concepts in a domain are values, not entities.
5. DDD is not about technology. It is a set of thinking tools for
   understanding and modeling complex business domains.
