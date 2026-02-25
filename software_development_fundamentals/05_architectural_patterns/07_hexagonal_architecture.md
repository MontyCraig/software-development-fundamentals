# Hexagonal Architecture (Ports and Adapters)

## Overview and Intent

Hexagonal Architecture, also known as Ports and Adapters, structures an
application so that its core business logic is completely isolated from
external concerns -- databases, user interfaces, messaging systems, and
third-party services. The core defines abstract "ports" (interfaces) that
describe what it needs, and concrete "adapters" implement those ports to
connect the core to the outside world.

The intent is to make the core logic testable, portable, and independent
of infrastructure decisions. A database can be swapped from relational to
document-based by writing a new adapter; the core logic does not change.
A command-line interface can replace a web interface by writing a new
driving adapter; again, the core remains untouched.

This pattern was introduced by Alistair Cockburn and is a direct
predecessor to Clean Architecture. The hexagonal shape is symbolic -- it
represents the idea that the application has many "sides" (ports) through
which the outside world can interact with it, rather than just a top and
bottom as in layered architecture.

## Structure and Components

### Component Diagram

```
            Driving Side                          Driven Side
         (Primary Actors)                     (Secondary Actors)
                                                      
    +----------------+                    +-------------------+
    | Web Adapter    |                    | Database Adapter  |
    | (REST Handler) +---+          +-----+ (SQL Repository)  |
    +----------------+   |          |     +-------------------+
                         |          |
    +----------------+   |          |     +-------------------+
    | CLI Adapter    +---+    +-----+-----+ Message Adapter   |
    +----------------+   |    |     |     | (Queue Publisher)  |
                         v    v     v     +-------------------+
                    +----+----+----+----+
                    |                   |      +-------------------+
                    |   APPLICATION     |      | File Adapter      |
                    |   CORE            +------+ (Storage Writer)  |
                    |                   |      +-------------------+
                    |  +-------------+  |
    +----------+    |  | Domain      |  |     +-------------------+
    | Test     +----+  | Model       |  +-----+ External API      |
    | Adapter  |    |  +-------------+  |     | Adapter           |
    +----------+    |  +-------------+  |     | (HTTP Client)     |
                    |  | Use Cases   |  |     +-------------------+
                    |  +-------------+  |
                    |  +-------------+  |
                    |  | Ports       |  |
                    |  | (Interfaces)|  |
                    |  +-------------+  |
                    +-------------------+
```

### Key Components

1. **Application Core**: Contains domain models, business rules, and use
   cases. Has zero dependencies on external frameworks or infrastructure.

2. **Port (Interface)**: An abstract contract defined by the core that
   describes an interaction point. Driving ports define what the core
   offers; driven ports define what the core needs.

3. **Driving Adapter (Primary)**: Translates external input (HTTP requests,
   CLI commands, test calls) into calls on the core's driving ports.

4. **Driven Adapter (Secondary)**: Implements the core's driven ports by
   connecting to external systems (databases, APIs, message queues).

5. **Dependency Injection**: The mechanism that wires adapters to ports at
   application startup, keeping the core unaware of concrete implementations.

## How It Works

```
  User Action (e.g., "Create Order")
       |
       v
  +----+-----+
  | Driving   |  1. Receive external input
  | Adapter   |  2. Translate to core method call
  | (Web API) |
  +----+------+
       |
       | calls Driving Port
       v
  +----+-----+
  | Use Case  |  3. Execute business logic
  | (Core)    |  4. Call Driven Port for data access
  +----+------+
       |
       | calls Driven Port (interface)
       v
  +----+------+
  | Driven    |  5. Translate to infrastructure call
  | Adapter   |  6. Execute against external system
  | (DB Repo) |
  +----+------+
       |
       v
  [Database]     7. Return raw data
       |
  (Adapter maps back to domain object)
       |
  (Core continues business logic)
       |
  (Driving Adapter formats response)
       |
       v
  Response sent to user
```

## Pseudocode Example

```
// --- Driven Port (Interface defined by the Core) ---

INTERFACE OrderRepository:
    FUNCTION Save(order: Order) -> Order
    FUNCTION FindById(orderId: String) -> Order
    FUNCTION FindByCustomer(customerId: String) -> List of Order
    FUNCTION Update(order: Order) -> Order
END INTERFACE

INTERFACE PaymentGateway:
    FUNCTION Charge(customerId: String, amount: Decimal) -> PaymentResult
    FUNCTION Refund(paymentId: String, amount: Decimal) -> RefundResult
END INTERFACE

INTERFACE NotificationPort:
    FUNCTION Send(recipient: String, subject: String, body: String) -> Boolean
END INTERFACE

// --- Driving Port (Interface the Core offers) ---

INTERFACE OrderManagement:
    FUNCTION PlaceOrder(command: PlaceOrderCommand) -> OrderResult
    FUNCTION CancelOrder(command: CancelOrderCommand) -> CancelResult
    FUNCTION GetOrder(query: GetOrderQuery) -> OrderView
END INTERFACE

// --- Domain Model (inside the Core) ---

STRUCTURE Order:
    id: String
    customerId: String
    items: List of OrderItem
    status: OrderStatus  // PENDING, CONFIRMED, SHIPPED, CANCELLED
    total: Decimal
    createdAt: DateTime
END STRUCTURE

ENUMERATION OrderStatus:
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
END ENUMERATION

FUNCTION Order.CalculateTotal() -> Decimal:
    SET sum = 0
    FOR EACH item IN this.items DO
        SET sum = sum + (item.unitPrice * item.quantity)
    END FOR
    RETURN sum
END FUNCTION

FUNCTION Order.Cancel() -> Boolean:
    IF this.status = SHIPPED OR this.status = DELIVERED THEN
        RETURN FALSE  // Cannot cancel shipped orders
    END IF
    SET this.status = CANCELLED
    RETURN TRUE
END FUNCTION

// --- Use Case (Application Core) ---

STRUCTURE PlaceOrderUseCase IMPLEMENTS OrderManagement:
    orderRepo: OrderRepository       // Driven port
    paymentGateway: PaymentGateway   // Driven port
    notifier: NotificationPort       // Driven port
END STRUCTURE

FUNCTION PlaceOrderUseCase.PlaceOrder(command: PlaceOrderCommand) -> OrderResult:
    // Pure business logic -- no infrastructure knowledge
    IF Length(command.items) = 0 THEN
        RETURN OrderResult.Failure("Order must have at least one item")
    END IF

    SET order = NEW Order
    SET order.id = GenerateId()
    SET order.customerId = command.customerId
    SET order.items = command.items
    SET order.status = PENDING
    SET order.createdAt = Now()
    SET order.total = order.CalculateTotal()

    IF order.total <= 0 THEN
        RETURN OrderResult.Failure("Order total must be positive")
    END IF

    // Use driven ports -- the core does not know HOW these work
    SET savedOrder = this.orderRepo.Save(order)

    SET paymentResult = this.paymentGateway.Charge(
        command.customerId, order.total
    )

    IF NOT paymentResult.success THEN
        SET savedOrder.status = CANCELLED
        this.orderRepo.Update(savedOrder)
        RETURN OrderResult.Failure("Payment failed: " + paymentResult.reason)
    END IF

    SET savedOrder.status = CONFIRMED
    this.orderRepo.Update(savedOrder)

    this.notifier.Send(
        command.customerEmail,
        "Order Confirmed",
        "Your order " + savedOrder.id + " has been confirmed."
    )

    RETURN OrderResult.Success(savedOrder)
END FUNCTION

// --- Driving Adapter (Web API) ---

FUNCTION WebAdapter.HandlePlaceOrder(httpRequest: HttpRequest) -> HttpResponse:
    // Translate HTTP request to domain command
    SET body = ParseBody(httpRequest)
    SET command = NEW PlaceOrderCommand
    SET command.customerId = body["customerId"]
    SET command.customerEmail = body["email"]
    SET command.items = MapItems(body["items"])

    // Call the driving port
    SET result = this.orderManagement.PlaceOrder(command)

    // Translate domain result to HTTP response
    IF result.isSuccess THEN
        RETURN HttpResponse(201, Serialize(result.order))
    ELSE
        RETURN HttpResponse(400, Serialize({error: result.errorMessage}))
    END IF
END FUNCTION

// --- Driven Adapter (Database Repository) ---

STRUCTURE SqlOrderRepository IMPLEMENTS OrderRepository:
    connection: DatabaseConnection
END STRUCTURE

FUNCTION SqlOrderRepository.Save(order: Order) -> Order:
    SET query = "INSERT INTO orders (id, customer_id, status, total, created_at) VALUES (:id, :cid, :status, :total, :created)"
    this.connection.Execute(query, {
        id: order.id,
        cid: order.customerId,
        status: order.status.ToString(),
        total: order.total,
        created: order.createdAt
    })
    FOR EACH item IN order.items DO
        SET itemQuery = "INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES (:oid, :pid, :qty, :price)"
        this.connection.Execute(itemQuery, {
            oid: order.id, pid: item.productId,
            qty: item.quantity, price: item.unitPrice
        })
    END FOR
    RETURN order
END FUNCTION

// --- Test Adapter (In-Memory Repository for Testing) ---

STRUCTURE InMemoryOrderRepository IMPLEMENTS OrderRepository:
    orders: Map of String to Order
END STRUCTURE

FUNCTION InMemoryOrderRepository.Save(order: Order) -> Order:
    SET this.orders[order.id] = order
    RETURN order
END FUNCTION

FUNCTION InMemoryOrderRepository.FindById(orderId: String) -> Order:
    RETURN this.orders[orderId]  // Returns NULL if not found
END FUNCTION

// --- Application Wiring (Composition Root) ---

FUNCTION Bootstrap() -> Application:
    // Production wiring
    SET dbConnection = ConnectDatabase("production-db-url")
    SET orderRepo = NEW SqlOrderRepository(dbConnection)
    SET paymentGateway = NEW StripePaymentAdapter("api-key")
    SET notifier = NEW EmailNotificationAdapter("smtp-config")

    SET orderUseCase = NEW PlaceOrderUseCase(orderRepo, paymentGateway, notifier)

    SET webAdapter = NEW WebAdapter(orderUseCase)
    RETURN NEW Application(webAdapter, port: 8080)
END FUNCTION

FUNCTION BootstrapForTests() -> PlaceOrderUseCase:
    // Test wiring -- no real infrastructure
    SET orderRepo = NEW InMemoryOrderRepository()
    SET paymentGateway = NEW FakePaymentGateway(alwaysSucceed: TRUE)
    SET notifier = NEW FakeNotifier()
    RETURN NEW PlaceOrderUseCase(orderRepo, paymentGateway, notifier)
END FUNCTION
```

## When to Use

- **Applications with rich business logic** that must be tested
  independently of infrastructure.
- **Systems likely to change infrastructure** (switching databases,
  switching cloud providers, replacing third-party services).
- **Teams practicing test-driven development** who need fast, reliable
  unit tests without infrastructure dependencies.
- **Long-lived applications** where infrastructure decisions made today
  may not be appropriate five years from now.
- **Multiple delivery mechanisms** (web API, CLI, batch processing) that
  share the same core business logic.

## When NOT to Use

- **Simple CRUD applications** with little business logic, where the
  indirection of ports and adapters adds more ceremony than value.
- **Throwaway prototypes** where speed of development matters more than
  architectural purity.
- **Extremely performance-sensitive systems** where the abstraction layers
  introduce measurable overhead.
- **Small teams on tight deadlines** who cannot afford the upfront
  investment in defining ports and adapters.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Financial trading | Order matching engine | Core logic must be testable and infrastructure-independent |
| Healthcare | Clinical decision support | Business rules must work regardless of data source |
| Insurance | Policy rating engine | Rating logic shared across web, batch, and API channels |
| Logistics | Route optimization | Algorithm testable without real maps or GPS services |
| SaaS platform | Multi-tenant application | Different tenants may use different storage backends |

## Trade-offs

### Advantages

- **Testability**: Core logic is trivially testable with fake adapters.
- **Infrastructure independence**: Swap databases, APIs, or UIs freely.
- **Clear boundaries**: Ports define explicit contracts between inside/outside.
- **Longevity**: Core logic outlives infrastructure decisions.
- **Multiple entry points**: Same logic served via web, CLI, or tests.

### Disadvantages

- **Indirection overhead**: More interfaces and classes to manage.
- **Learning curve**: The port/adapter concept requires training.
- **Over-engineering risk**: Simple CRUD apps do not benefit.
- **Mapping cost**: Data must be mapped between adapter and domain forms.

## Comparison with Related Patterns

| Dimension | Hexagonal | Layered | Clean Architecture |
|-----------|-----------|---------|-------------------|
| Dependency direction | Outside-in | Top-down | Outside-in (concentric) |
| Core isolation | High | Medium | High |
| Number of boundaries | 2 (driving/driven) | N (per layer) | 4 (concentric circles) |
| Testability | Excellent | Good | Excellent |
| Complexity | Medium | Low | Medium-High |

## Evolution and Variations

- **Original Hexagonal**: Cockburn's formulation with driving and driven
  ports. The hexagonal shape is metaphorical.
- **Onion Architecture**: Jeffrey Palermo's variant that adds explicit
  concentric layers (domain model, domain services, application services).
- **Clean Architecture**: Robert Martin's synthesis that formalizes the
  concentric circle model with entities, use cases, and interface adapters.
- **Functional Core, Imperative Shell**: Functional programming variant
  where the core is pure functions and the shell handles side effects.

## Key Takeaways

1. The core principle is dependency inversion: the core defines what it
   needs (ports), and the outside world provides it (adapters).
2. Driving adapters call into the core; driven adapters are called by the
   core. Both connect through abstract ports.
3. The composition root (application startup) is the only place that knows
   about concrete adapter implementations.
4. Test adapters (fakes, stubs) are first-class citizens, not afterthoughts.
   They are the primary reason the pattern exists.
5. Hexagonal architecture shines when business logic is complex and
   infrastructure is likely to change. For simple CRUD, it is overkill.
