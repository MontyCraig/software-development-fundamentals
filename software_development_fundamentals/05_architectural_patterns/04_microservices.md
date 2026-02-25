# Microservices Architecture

## Overview and Intent

Microservices architecture structures an application as a collection of small,
independently deployable services, each running in its own process and
communicating over lightweight protocols. Each service is built around a
specific business capability, owns its own data, and can be developed, deployed,
and scaled independently.

The intent is to enable large organizations to deliver software faster by
allowing autonomous teams to own, develop, and operate individual services
without coordinating deployments across the entire system. Microservices
trade the simplicity of a monolith for the flexibility of independent
evolution, technology heterogeneity, and fine-grained scaling.

This pattern is not a silver bullet. It introduces significant complexity in
networking, data consistency, observability, and operations. It is best suited
for organizations with mature engineering practices, dedicated platform teams,
and genuine scaling or team-coordination pain points that a monolith cannot solve.

## Structure and Components

### Component Diagram

```
                         +------------------+
            +----------->| API Gateway      |<-----------+
            |            | (Routing, Auth,  |            |
            |            |  Rate Limiting)  |            |
            |            +--------+---------+            |
            |                     |                      |
     +------+------+    +--------+--------+    +--------+--------+
     | User        |    | Order           |    | Inventory       |
     | Service     |    | Service         |    | Service         |
     |             |    |                 |    |                 |
     | +--------+  |    | +--------+     |    | +--------+     |
     | | Logic   | |    | | Logic   |    |    | | Logic   |    |
     | +--------+  |    | +--------+     |    | +--------+     |
     | | Data    | |    | | Data    |    |    | | Data    |    |
     | | Store   | |    | | Store   |    |    | | Store   |    |
     | +--------+  |    | +--------+     |    | +--------+     |
     +------+------+    +--------+--------+    +--------+--------+
            |                    |                       |
            +----------+---------+-----------+-----------+
                       |                     |
              +--------+--------+   +--------+--------+
              | Service Mesh    |   | Message Broker   |
              | (Discovery,     |   | (Async Events)   |
              |  Load Balance)  |   |                  |
              +-----------------+   +------------------+
```

### Key Components

1. **API Gateway**: Single entry point for external clients. Routes requests
   to appropriate services, handles cross-cutting concerns like authentication,
   rate limiting, and request transformation.

2. **Individual Services**: Autonomous units of business functionality. Each
   service has its own codebase, data store, and deployment pipeline.

3. **Service Mesh / Service Discovery**: Infrastructure layer that manages
   service-to-service communication, load balancing, circuit breaking, and
   mutual authentication.

4. **Message Broker**: Enables asynchronous communication between services
   through events or messages, decoupling producers from consumers.

5. **Distributed Tracing**: Observability infrastructure that tracks requests
   as they flow across multiple services.

## How It Works

```
  Client Request: "Place Order"
       |
       v
  +----+------+
  | API       |  1. Authenticate and route
  | Gateway   |
  +----+------+
       |
       v
  +----+------+
  | Order     |  2. Validate order data
  | Service   |  3. Call User Service (sync) to verify customer
  +----+------+  4. Call Inventory Service (sync) to check stock
       |
       |--- sync call ---> User Service ---> User DB
       |                        |
       |<--- user data ---------+
       |
       |--- sync call ---> Inventory Service ---> Inventory DB
       |                        |
       |<--- stock status ------+
       |
       |  5. Save order to Order DB
       |  6. Publish "OrderPlaced" event
       |
       +--- async event ---> Payment Service
       |                     (processes payment independently)
       +--- async event ---> Notification Service
       |                     (sends confirmation email)
       +--- async event ---> Analytics Service
                             (records business metrics)
```

## Pseudocode Example

```
// --- API Gateway ---

FUNCTION GatewayRoute(request: Request) -> Response:
    // Cross-cutting: authentication
    SET token = request.headers["Authorization"]
    SET identity = AuthService.Validate(token)
    IF identity IS NULL THEN
        RETURN Response.Unauthorized("Invalid token")
    END IF

    // Cross-cutting: rate limiting
    IF RateLimiter.IsExceeded(identity.clientId) THEN
        RETURN Response.TooManyRequests("Rate limit exceeded")
    END IF

    // Route to appropriate service
    SET serviceUrl = ServiceRegistry.Lookup(request.path)
    SET enrichedRequest = AddHeaders(request, {
        "X-Client-Id": identity.clientId,
        "X-Trace-Id": GenerateTraceId()
    })

    RETURN ForwardRequest(serviceUrl, enrichedRequest)
END FUNCTION

// --- Order Service ---

FUNCTION CreateOrder(request: Request) -> Response:
    SET orderData = ParseBody(request.body)
    SET traceId = request.headers["X-Trace-Id"]

    // Step 1: Validate customer exists (synchronous call to User Service)
    SET userResponse = ServiceClient.Get(
        url: ServiceRegistry.Lookup("user-service") + "/users/" + orderData.customerId,
        headers: {"X-Trace-Id": traceId},
        timeout: 3000,
        retries: 2
    )
    IF userResponse.status != 200 THEN
        RETURN Response.BadRequest("Invalid customer")
    END IF

    // Step 2: Check inventory (synchronous call with circuit breaker)
    FOR EACH item IN orderData.items DO
        SET stockResponse = CircuitBreaker.Execute("inventory-service", FUNCTION():
            RETURN ServiceClient.Get(
                url: ServiceRegistry.Lookup("inventory-service") +
                     "/stock/" + item.productId,
                headers: {"X-Trace-Id": traceId},
                timeout: 2000
            )
        END FUNCTION)

        IF stockResponse.status != 200 OR stockResponse.body.available < item.quantity THEN
            RETURN Response.Conflict("Insufficient stock for " + item.productId)
        END IF
    END FOR

    // Step 3: Save order to local database
    SET order = NEW Order
    SET order.id = GenerateUUID()
    SET order.customerId = orderData.customerId
    SET order.items = orderData.items
    SET order.status = "PENDING"
    SET order.createdAt = Now()
    SET order.total = CalculateTotal(orderData.items)

    OrderDatabase.Save(order)

    // Step 4: Publish event for asynchronous processing
    SET event = NEW OrderPlacedEvent
    SET event.orderId = order.id
    SET event.customerId = order.customerId
    SET event.items = order.items
    SET event.total = order.total
    SET event.timestamp = Now()

    MessageBroker.Publish("orders.placed", event)

    RETURN Response.Created({orderId: order.id, status: "PENDING"})
END FUNCTION

// --- Service Discovery ---

FUNCTION ServiceRegistry.Lookup(serviceName: String) -> String:
    SET instances = Registry.GetHealthyInstances(serviceName)
    IF Length(instances) = 0 THEN
        RAISE ServiceUnavailable(serviceName + " has no healthy instances")
    END IF
    // Load balance across instances
    RETURN LoadBalancer.ChooseInstance(instances, strategy: "round-robin")
END FUNCTION

// --- Circuit Breaker ---

FUNCTION CircuitBreaker.Execute(serviceName: String, operation: Function) -> Response:
    SET breaker = this.breakers[serviceName]

    IF breaker.state = "OPEN" THEN
        IF Now() - breaker.openedAt > breaker.resetTimeout THEN
            SET breaker.state = "HALF_OPEN"
        ELSE
            RETURN Response.ServiceUnavailable("Circuit open for " + serviceName)
        END IF
    END IF

    TRY
        SET result = operation()
        breaker.RecordSuccess()
        IF breaker.state = "HALF_OPEN" THEN
            SET breaker.state = "CLOSED"
        END IF
        RETURN result
    CATCH error AS TimeoutError
        breaker.RecordFailure()
        IF breaker.failureCount >= breaker.threshold THEN
            SET breaker.state = "OPEN"
            SET breaker.openedAt = Now()
        END IF
        RAISE error
    END TRY
END FUNCTION

// --- Payment Service (Event Consumer) ---

FUNCTION PaymentEventHandler.OnOrderPlaced(event: OrderPlacedEvent):
    SET payment = NEW Payment
    SET payment.orderId = event.orderId
    SET payment.amount = event.total
    SET payment.status = "PROCESSING"

    PaymentDatabase.Save(payment)

    SET result = PaymentGateway.Charge(event.customerId, event.total)
    IF result.success THEN
        SET payment.status = "COMPLETED"
        MessageBroker.Publish("payments.completed", {orderId: event.orderId})
    ELSE
        SET payment.status = "FAILED"
        MessageBroker.Publish("payments.failed", {
            orderId: event.orderId,
            reason: result.failureReason
        })
    END IF
    PaymentDatabase.Update(payment)
END FUNCTION
```

## When to Use

- **Large organizations (20+ developers)** with multiple teams that need
  independent development and deployment cycles.
- **Systems with distinct scaling requirements** where some components need
  10x the resources of others.
- **Polyglot environments** where different services benefit from different
  technology stacks or data stores.
- **Continuous deployment requirements** where teams must release features
  independently multiple times per day.
- **Fault isolation needs** where a failure in one business area must not
  cascade to others.

## When NOT to Use

- **Small teams (under 10)** where the operational overhead of distributed
  systems outweighs the organizational benefits.
- **New or poorly understood domains** where service boundaries are unclear.
  Start with a monolith and split when boundaries stabilize.
- **Systems requiring strong consistency** across multiple business areas.
  Distributed transactions are complex and fragile.
- **Organizations without mature DevOps practices** including automated
  testing, CI/CD, monitoring, and incident response.
- **Simple applications** where the overhead of service discovery, API
  gateways, and distributed tracing is not justified.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| E-commerce | Large online marketplace | Independent scaling of search, catalog, checkout, payments |
| Streaming media | Video platform | Separate services for transcoding, recommendation, delivery |
| Financial services | Trading platform | Independent deployment of pricing, risk, execution engines |
| Social media | Content platform | Separate services for feed, messaging, notifications, ads |
| Logistics | Shipping management | Independent tracking, routing, billing, and scheduling |

## Trade-offs

### Advantages

- **Independent deployment**: Teams release on their own schedule.
- **Technology diversity**: Each service can use the best-fit stack.
- **Granular scaling**: Scale only the services that need it.
- **Fault isolation**: One service failure does not crash the system.
- **Team autonomy**: Small teams own services end-to-end.

### Disadvantages

- **Distributed system complexity**: Network latency, partial failures,
  eventual consistency.
- **Operational overhead**: Dozens or hundreds of services to deploy,
  monitor, and troubleshoot.
- **Data consistency challenges**: No cross-service ACID transactions.
- **Testing difficulty**: Integration and end-to-end tests are complex.
- **Service coordination**: Versioning, backward compatibility, and
  contract management require discipline.

## Comparison with Related Patterns

| Dimension | Microservices | SOA | Monolith |
|-----------|--------------|-----|----------|
| Service size | Small, focused | Large, enterprise-wide | One unit |
| Communication | Lightweight protocols | ESB, formal contracts | In-process |
| Data ownership | Per-service databases | Shared or partitioned | Shared database |
| Governance | Decentralized | Centralized | N/A |
| Deployment | Independent | Coordinated | Single |

## Evolution and Variations

- **Service Mesh**: Infrastructure layer that handles service-to-service
  communication concerns (load balancing, encryption, observability)
  transparently, without application code changes.
- **Micro-frontends**: Extends the microservices concept to the user
  interface, allowing teams to own full vertical slices including the UI.
- **Nanoservices**: Extremely small services (often a single function).
  Generally considered an anti-pattern due to excessive overhead.
- **Macro-services**: Larger services that own broader business domains.
  A practical middle ground for organizations transitioning from monoliths.

## Key Takeaways

1. Microservices are an organizational pattern as much as a technical one.
   They work best when service boundaries align with team boundaries.
2. Start with a monolith and extract services when you have clear evidence
   of the need. Premature decomposition creates distributed monoliths.
3. Invest heavily in infrastructure: CI/CD, service discovery, distributed
   tracing, centralized logging, and automated testing.
4. Design for failure. Every inter-service call can fail. Use circuit
   breakers, retries with backoff, timeouts, and fallback strategies.
5. Data consistency across services requires careful design. Prefer
   eventual consistency with compensating transactions (sagas) over
   distributed ACID transactions.
