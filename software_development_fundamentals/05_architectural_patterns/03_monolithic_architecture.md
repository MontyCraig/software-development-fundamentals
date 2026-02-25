# Monolithic Architecture

## Overview and Intent

A monolithic architecture packages all of an application's functionality into a
single deployable unit. The entire system -- user interface, business logic, data
access, and background processing -- is built, tested, and deployed as one
artifact. Despite its reputation as an "outdated" approach, the monolith remains
the most practical starting point for the majority of new projects.

The intent is simplicity: one codebase, one build pipeline, one deployment, one
runtime process. This minimizes operational overhead and allows small teams to
move quickly. When done well, a monolith is not a "big ball of mud" but a
well-structured application with clear internal boundaries.

The "modular monolith" variation has emerged as a middle ground between a
traditional monolith and microservices. It maintains a single deployment unit
but enforces strict module boundaries internally, giving teams the option to
extract modules into separate services later if needed.

## Structure and Components

### Component Diagram

```
+================================================================+
|                     MONOLITHIC APPLICATION                       |
|                                                                  |
|  +------------------+  +------------------+  +----------------+  |
|  |   Module A       |  |   Module B       |  |   Module C     |  |
|  |  (User Mgmt)     |  |  (Orders)        |  |  (Inventory)   |  |
|  |                   |  |                   |  |                |  |
|  | +------+ +------+ |  | +------+ +------+ |  | +------+     |  |
|  | |  UI  | |Logic | |  | |  UI  | |Logic | |  | |Logic |     |  |
|  | +------+ +------+ |  | +------+ +------+ |  | +------+     |  |
|  | +------+          |  | +------+          |  | +------+     |  |
|  | | Data |          |  | | Data |          |  | | Data |     |  |
|  | +------+          |  | +------+          |  | +------+     |  |
|  +--------+----------+  +--------+----------+  +-------+------+  |
|           |                      |                      |        |
|  +--------+----------------------+----------------------+-----+  |
|  |                    SHARED DATABASE                         |  |
|  +------------------------------------------------------------+  |
+================================================================+
               |
               | Single deployment artifact
               v
        +--------------+
        | App Server   |
        +--------------+
```

### Key Components

1. **Application Module**: A logically distinct feature area (users, orders,
   inventory). In a well-structured monolith, each module encapsulates its
   own UI routes, business logic, and data access.

2. **Shared Infrastructure**: Cross-cutting services like authentication,
   logging, configuration, and connection pooling shared by all modules.

3. **Shared Database**: A single database instance used by all modules.
   This simplifies transactions but creates coupling at the data level.

4. **Build Pipeline**: One pipeline that compiles, tests, and packages
   the entire application into a single artifact.

5. **Runtime Process**: One operating system process (or a small cluster
   of identical processes behind a load balancer) that serves all requests.

## How It Works

```
  Request arrives
       |
       v
+------+-------+
| Router /     |   1. Route maps URL/command to the correct module
| Dispatcher    |
+------+-------+
       |
       v
+------+-------+
| Module       |   2. Module handles the request using its own
| (e.g. Orders)|      business logic and data access code
+------+-------+
       |
       v
+------+-------+
| Shared DB    |   3. All modules read/write to the same database
+------+-------+   4. Transactions can span modules easily
       |
       v
  Response sent
```

Detailed flow:

1. An incoming request arrives at the application's single entry point.
2. A router or dispatcher examines the request and forwards it to the
   appropriate module.
3. The module processes the request: validates input, applies business
   rules, reads/writes data.
4. Because all modules share one database, cross-module queries and
   transactions are straightforward.
5. The module produces a response, which flows back through the router
   to the client.
6. Background tasks (email sending, report generation) run in the same
   process, often triggered by in-process events or scheduled timers.

## Pseudocode Example

```
// --- Application Entry Point ---

FUNCTION Main():
    SET config = LoadConfiguration("app.config")
    SET database = ConnectDatabase(config.databaseUrl)
    
    // Initialize all modules with shared dependencies
    SET userModule = NEW UserModule(database)
    SET orderModule = NEW OrderModule(database)
    SET inventoryModule = NEW InventoryModule(database)
    
    SET router = NEW Router
    router.Register("/users", userModule)
    router.Register("/orders", orderModule)
    router.Register("/inventory", inventoryModule)
    
    SET server = NEW Server(config.port, router)
    server.Start()
    LOG "Application started on port " + config.port
END FUNCTION

// --- Router ---

FUNCTION Router.HandleRequest(request: Request) -> Response:
    SET module = this.FindModule(request.path)
    IF module IS NULL THEN
        RETURN Response.NotFound("No handler for " + request.path)
    END IF
    
    TRY
        SET response = module.Handle(request)
        RETURN response
    CATCH error AS ApplicationError
        LOG "Error handling request: " + error.message
        RETURN Response.ServerError(error.message)
    END TRY
END FUNCTION

// --- Order Module ---

FUNCTION OrderModule.Handle(request: Request) -> Response:
    IF request.method = "POST" AND request.path = "/orders" THEN
        RETURN this.CreateOrder(request)
    ELSE IF request.method = "GET" AND request.path = "/orders" THEN
        RETURN this.ListOrders(request)
    END IF
    RETURN Response.NotFound("Unknown order endpoint")
END FUNCTION

FUNCTION OrderModule.CreateOrder(request: Request) -> Response:
    SET orderData = ParseBody(request.body)
    
    // Validate input
    IF orderData.items IS EMPTY THEN
        RETURN Response.BadRequest("Order must contain at least one item")
    END IF
    
    // Cross-module call within the same process
    FOR EACH item IN orderData.items DO
        SET available = this.inventoryModule.CheckStock(item.productId, item.quantity)
        IF NOT available THEN
            RETURN Response.Conflict("Insufficient stock for product " + item.productId)
        END IF
    END FOR
    
    // Transaction spans order and inventory tables
    SET transaction = this.database.BeginTransaction()
    TRY
        SET order = NEW Order
        SET order.customerId = orderData.customerId
        SET order.status = "PENDING"
        SET order.createdAt = Now()
        
        SET savedOrder = OrderRepository.Save(order, transaction)
        
        FOR EACH item IN orderData.items DO
            SET orderItem = NEW OrderItem
            SET orderItem.orderId = savedOrder.id
            SET orderItem.productId = item.productId
            SET orderItem.quantity = item.quantity
            SET orderItem.unitPrice = item.price
            OrderItemRepository.Save(orderItem, transaction)
            
            // Reserve inventory in the same transaction
            InventoryRepository.DecrementStock(item.productId, item.quantity, transaction)
        END FOR
        
        SET savedOrder.total = CalculateTotal(orderData.items)
        OrderRepository.Update(savedOrder, transaction)
        
        transaction.Commit()
        
        // In-process event for background tasks
        EventBus.Publish(NEW OrderCreatedEvent(savedOrder))
        
        RETURN Response.Created(savedOrder)
    CATCH error
        transaction.Rollback()
        RAISE error
    END TRY
END FUNCTION

// --- Modular Monolith: Module Boundary Enforcement ---

STRUCTURE ModuleBoundary:
    allowedDependencies: List of ModuleName
    publicInterface: List of FunctionSignature
    internalComponents: List of Component
END STRUCTURE

FUNCTION EnforceModuleBoundary(caller: Module, target: Module, operation: String):
    IF NOT Contains(caller.boundary.allowedDependencies, target.name) THEN
        RAISE BoundaryViolation(
            caller.name + " is not allowed to call " + target.name
        )
    END IF
    IF NOT Contains(target.boundary.publicInterface, operation) THEN
        RAISE BoundaryViolation(
            operation + " is not part of " + target.name + " public interface"
        )
    END IF
END FUNCTION
```

## When to Use

- **Early-stage projects and startups** where speed of development matters
  more than scalability. Ship fast, refactor later.
- **Small teams (1-10 developers)** who benefit from a single codebase with
  simple debugging, testing, and deployment.
- **Systems with tightly coupled domains** where cross-domain transactions
  are frequent and consistency is critical.
- **Applications with moderate scale** that can be served by a single server
  or a small cluster behind a load balancer.
- **When operational simplicity is a priority**: one build, one deployment,
  one set of logs, one monitoring dashboard.

## When NOT to Use

- **Large teams (20+)** working on independent feature areas who would
  create constant merge conflicts in a shared codebase.
- **Systems requiring independent scaling** of different components (e.g.,
  CPU-intensive image processing versus IO-heavy API serving).
- **Polyglot requirements** where different parts of the system would
  benefit from different technology stacks.
- **High availability requirements** where a bug in one module should not
  crash the entire application.
- **Rapidly evolving systems** where different features have vastly
  different release cadences.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Content management | Blog or wiki platform | Moderate complexity, single team, shared content DB |
| Internal tools | Admin dashboard | Small user base, fast development priority |
| E-commerce startup | MVP online store | Quick to market, cross-domain transactions |
| Accounting | Bookkeeping application | Strong consistency needs, moderate scale |
| Education | Learning management system | Small team, integrated features, simple operations |

## Trade-offs

### Advantages

- **Simple development**: One codebase, one IDE project, one debug session.
- **Simple deployment**: Build once, deploy one artifact.
- **Simple testing**: End-to-end tests run in a single process.
- **Cross-module transactions**: ACID transactions span module boundaries.
- **Low operational overhead**: One process to monitor, one set of logs.
- **Low latency for in-process calls**: Module-to-module calls have no
  network overhead.

### Disadvantages

- **Deployment coupling**: A small change requires redeploying everything.
- **Scaling limitations**: Cannot scale modules independently.
- **Technology lock-in**: The entire application must use one tech stack.
- **Fault isolation**: A memory leak in one module crashes all modules.
- **Team scaling**: Large teams create bottlenecks and merge conflicts.
- **Startup time**: As the application grows, cold starts slow down.

## Comparison with Related Patterns

| Dimension | Monolithic | Modular Monolith | Microservices |
|-----------|-----------|-----------------|---------------|
| Deployment unit | Single | Single | Many |
| Module boundaries | Weak or none | Enforced interfaces | Network boundaries |
| Cross-module calls | In-process | In-process via interfaces | Network (latency, failure) |
| Data ownership | Shared database | Shared or partitioned | Independent databases |
| Team independence | Low | Medium | High |
| Operational complexity | Low | Low | High |

## Evolution and Variations

- **Big Ball of Mud**: The anti-pattern where internal structure degrades
  over time. Prevent with coding standards, code review, and static analysis.
- **Modular Monolith**: Enforces strict module boundaries with explicit
  public interfaces. Modules communicate only through defined contracts.
  This is the recommended starting point for most new projects.
- **Monolith-First Strategy**: Build a monolith, then extract services as
  clear boundaries and scaling needs emerge. Avoids premature distribution.
- **Distributed Monolith**: The worst-case outcome of a failed microservices
  migration -- distributed deployment with monolithic coupling. Avoid this
  by ensuring true independence before splitting.

## Key Takeaways

1. The monolith is not an anti-pattern. It is the right starting point for
   most projects and remains the right architecture for many systems
   throughout their lifetime.
2. A modular monolith gives you most of the organizational benefits of
   microservices without the operational cost of distributed systems.
3. The decision to move from monolith to microservices should be driven by
   concrete pain points (scaling, team coordination, deployment speed),
   not by industry trends.
4. Cross-module ACID transactions are a monolith superpower. If your
   domain requires frequent cross-boundary consistency, think twice
   before distributing.
5. Invest in internal structure (module boundaries, clear interfaces,
   dependency rules) early. A well-structured monolith is easy to split
   later; a tangled one is nearly impossible.
