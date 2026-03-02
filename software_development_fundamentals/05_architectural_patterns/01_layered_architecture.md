# Layered (N-Tier) Architecture

## Overview and Intent

The Layered Architecture pattern organizes a system into horizontal layers, each
of which provides a cohesive set of responsibilities and depends only on the
layer directly beneath it. The most common variant uses three or four tiers:
presentation, business logic, data access, and optionally a persistence layer.

This pattern is the workhorse of enterprise software. Its primary intent is to
achieve separation of concerns so that changes in one layer (such as replacing a
database engine) do not ripple through the entire system. By constraining
dependencies to flow in one direction -- top to bottom -- the architecture
becomes easier to reason about, test in isolation, and maintain over time.

Layered architectures can be "strict" (each layer may only call the layer
immediately below) or "relaxed" (a layer may bypass intermediate layers). Strict
layering maximizes isolation at the cost of pass-through boilerplate; relaxed
layering reduces ceremony but introduces tighter coupling.

## Structure and Components

### Component Diagram

```
+-------------------------------------------------------+
|                  PRESENTATION LAYER                    |
|  (User interface, API endpoints, view rendering)      |
+---------------------------+---------------------------+
                            |  calls down
                            v
+-------------------------------------------------------+
|                   BUSINESS LAYER                       |
|  (Domain logic, validation, orchestration, rules)     |
+---------------------------+---------------------------+
                            |  calls down
                            v
+-------------------------------------------------------+
|                 DATA ACCESS LAYER                      |
|  (Repositories, queries, data mapping, caching)       |
+---------------------------+---------------------------+
                            |  calls down
                            v
+-------------------------------------------------------+
|                  PERSISTENCE LAYER                     |
|  (Database, file system, external storage)            |
+-------------------------------------------------------+
```

### Key Components

1. **Presentation Layer**: Handles all user interaction. Accepts input, formats
   output, delegates work to the business layer. Contains no business rules.

2. **Business Layer**: Encapsulates domain logic, validation, calculations, and
   workflow orchestration. This is the intellectual core of the application.

3. **Data Access Layer**: Abstracts storage mechanics. Translates between
   domain objects and storage representations. Manages connections and queries.

4. **Persistence Layer**: The actual storage mechanism -- relational databases,
   document stores, file systems, or external services.

## How It Works

```
  Client Request
       |
       v
+------------------+
| Presentation     |  1. Parse and validate input format
| Layer            |  2. Delegate to business layer
+--------+---------+
         |
         v
+------------------+
| Business         |  3. Apply business rules
| Layer            |  4. Orchestrate operations
+--------+---------+  5. Return domain result
         |
         v
+------------------+
| Data Access      |  6. Translate domain query to storage query
| Layer            |  7. Execute against persistence
+--------+---------+  8. Map results back to domain objects
         |
         v
+------------------+
| Persistence      |  9. Store/retrieve raw data
| Layer            |
+------------------+
```

Step-by-step flow for a typical read operation:

1. The presentation layer receives a request (e.g., "get order #1234").
2. It performs input validation (is the ID well-formed?) and calls the
   business layer.
3. The business layer applies authorization rules (can this user see this
   order?) and calls the data access layer.
4. The data access layer constructs and executes a query against the
   persistence layer.
5. Raw data is mapped into a domain object and returned up the stack.
6. The business layer applies any post-processing rules.
7. The presentation layer formats the response and sends it to the client.

## Pseudocode Example

```
// --- Presentation Layer ---

FUNCTION HandleGetOrder(request: Request) -> Response:
    SET orderId = ParseInteger(request.pathParams["id"])
    IF orderId IS NULL THEN
        RETURN ErrorResponse(400, "Invalid order ID format")
    END IF

    SET result = OrderService.GetOrder(orderId)
    IF result.isError THEN
        RETURN ErrorResponse(result.errorCode, result.errorMessage)
    END IF

    SET viewModel = MapToOrderView(result.order)
    RETURN SuccessResponse(200, viewModel)
END FUNCTION

FUNCTION MapToOrderView(order: Order) -> OrderView:
    SET view = NEW OrderView
    SET view.id = order.id
    SET view.customerName = order.customer.fullName
    SET view.totalFormatted = FormatCurrency(order.total)
    SET view.itemCount = Length(order.items)
    SET view.status = order.status.displayName
    RETURN view
END FUNCTION

// --- Business Layer ---

FUNCTION OrderService.GetOrder(orderId: Integer) -> OrderResult:
    SET order = OrderRepository.FindById(orderId)
    IF order IS NULL THEN
        RETURN OrderResult.Error(404, "Order not found")
    END IF

    IF NOT AuthorizationService.CanViewOrder(CurrentUser(), order) THEN
        RETURN OrderResult.Error(403, "Access denied")
    END IF

    SET order.total = CalculateOrderTotal(order)
    SET order.estimatedDelivery = ShippingRules.EstimateDelivery(order)

    RETURN OrderResult.Success(order)
END FUNCTION

FUNCTION CalculateOrderTotal(order: Order) -> Decimal:
    SET subtotal = 0
    FOR EACH item IN order.items DO
        SET subtotal = subtotal + (item.price * item.quantity)
    END FOR

    SET discount = DiscountRules.Apply(order.customer, subtotal)
    SET tax = TaxRules.Calculate(subtotal - discount, order.shippingAddress)
    RETURN subtotal - discount + tax
END FUNCTION

// --- Data Access Layer ---

FUNCTION OrderRepository.FindById(orderId: Integer) -> Order:
    SET query = "SELECT * FROM orders WHERE id = :id"
    SET row = Database.ExecuteQuery(query, {id: orderId})

    IF row IS NULL THEN
        RETURN NULL
    END IF

    SET order = MapRowToOrder(row)
    SET order.items = OrderItemRepository.FindByOrderId(orderId)
    SET order.customer = CustomerRepository.FindById(row["customer_id"])
    RETURN order
END FUNCTION

FUNCTION MapRowToOrder(row: DataRow) -> Order:
    SET order = NEW Order
    SET order.id = row["id"]
    SET order.createdAt = row["created_at"]
    SET order.status = OrderStatus.FromCode(row["status_code"])
    SET order.shippingAddress = AddressMapper.FromColumns(row)
    RETURN order
END FUNCTION
```

## Common Anti-Patterns

### The Sinkhole Anti-Pattern

When a request passes through multiple layers without any of them adding
meaningful value, the architecture has too many layers for that operation.

```
// Anti-pattern: business layer just forwards to data layer
FUNCTION BusinessLayer.GetUserName(userId: Integer) -> String:
    RETURN DataAccessLayer.GetUserName(userId)   // No logic added
END FUNCTION
```

If more than 20 percent of requests are simple pass-throughs, consider
whether all layers are necessary for those particular operation paths.

### Architecture Drift

Over time, developers may bypass layer boundaries for convenience. Common
symptoms include presentation layer code that directly queries the database,
business logic embedded in stored procedures at the persistence layer, and
data access objects that contain validation rules. Prevent drift with
automated dependency checks, code reviews, and clear module boundaries.

## When to Use

- **Small to medium applications** where a straightforward structure keeps the
  team productive without over-engineering.
- **Teams new to architecture patterns** who benefit from a well-understood,
  low-ceremony structure.
- **Systems with clear horizontal responsibilities** where presentation,
  business logic, and data access are distinct concerns.
- **Applications where a single deployment unit** is acceptable and deployment
  independence is not a priority.
- **Prototypes and MVPs** where development speed is prioritized over
  long-term scalability.

## When NOT to Use

- **High-scale systems requiring independent scaling** of different functional
  areas. Layers scale together, not independently.
- **Systems with complex cross-cutting concerns** that do not fit neatly into
  horizontal layers (e.g., analytics pipelines).
- **Large teams (20+)** working on the same codebase. Layer boundaries are too
  coarse to prevent merge conflicts and coordination overhead.
- **Real-time or event-driven workloads** where the synchronous call-down
  model adds unacceptable latency.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Corporate intranet | Employee self-service portal | Moderate complexity, small team, well-defined layers |
| E-commerce | Small online store | Clear separation: storefront, order logic, product catalog |
| Banking | Branch operations tool | Regulatory auditability benefits from strict layering |
| Healthcare | Patient records viewer | Data access isolation supports compliance requirements |
| Education | Course management system | Simple CRUD with moderate business rules |

## Trade-offs

### Advantages

- **Simplicity**: Easy to understand, teach, and implement.
- **Separation of concerns**: Changes to UI do not affect business logic.
- **Testability at the layer level**: Each layer can be tested with mocks for
  the layer below.
- **Well-understood**: Most developers have worked with this pattern.
- **Tooling support**: Many frameworks assume and support this structure.

### Disadvantages

- **Monolithic deployment**: All layers deploy as one unit.
- **Performance overhead**: Data must pass through every layer, even for
  simple pass-through operations (the "sinkhole" anti-pattern).
- **Rigid structure**: Cross-cutting concerns like logging and caching fit
  awkwardly into the layer model.
- **Coupling risk**: Without discipline, business logic leaks into
  presentation or data access layers.
- **Limited scalability**: Cannot scale layers independently.

## Comparison with Related Patterns

| Dimension | Layered | Hexagonal | Clean |
|-----------|---------|-----------|-------|
| Dependency direction | Top-down only | Outside-in via ports | Outside-in via boundaries |
| Core isolation | Medium | High | High |
| Infrastructure swap | Requires data layer changes | Adapter swap only | Adapter swap only |
| Complexity | Low | Medium | Medium |
| Typical use case | CRUD applications | Domain-rich applications | Complex business systems |

## Evolution and Variations

- **Strict vs. Relaxed Layering**: Strict requires each layer to call only
  the layer immediately below. Relaxed allows skipping layers for performance.
- **Closed vs. Open Layers**: Closed layers enforce that requests must pass
  through every layer. Open layers allow bypass for cross-cutting concerns.
- **Modular Monolith**: Combines layered architecture with vertical slicing
  by feature module, giving some benefits of microservices within a single
  deployment unit.
- **Onion Architecture**: A precursor to hexagonal and clean architectures
  that inverts the dependency direction of the traditional layered model.
- **N-Tier Deployment**: Each layer runs on separate physical or virtual
  machines, enabling independent scaling of each tier. Common in legacy
  enterprise deployments.

## Key Takeaways

1. Layered architecture is the default starting point for most applications.
   It is simple, well-understood, and sufficient for many use cases.
2. Strict layering prevents shortcut coupling but adds pass-through cost.
3. The "sinkhole anti-pattern" (layers that simply forward calls without
   adding value) is a sign that the architecture may be too granular.
4. Layered architecture works best for small-to-medium teams building
   applications with clear horizontal concerns and modest scale needs.
5. When you outgrow layered architecture, consider hexagonal or clean
   architecture as the next step, as they preserve layering benefits while
   adding infrastructure independence.
