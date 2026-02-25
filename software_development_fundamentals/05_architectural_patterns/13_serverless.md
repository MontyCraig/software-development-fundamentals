# Serverless Architecture

## Overview and Intent

Serverless architecture is a cloud execution model where the cloud provider
dynamically manages the allocation and provisioning of servers. Application
logic is decomposed into individual functions that are triggered by events,
execute in ephemeral containers, and are billed per invocation rather than per
server-hour. The developer writes only business logic; the platform handles
scaling, availability, and infrastructure management.

The intent is to eliminate operational overhead. There are no servers to
provision, patch, or scale. Functions scale automatically from zero to thousands
of concurrent instances and back to zero, with no idle cost. This model is
particularly suited to workloads that are sporadic, event-driven, or
unpredictable in volume.

Despite the name, servers do exist -- they are simply managed entirely by the
cloud provider and invisible to the developer. The "serverless" label refers
to the developer's experience, not the physical reality. The key trade-off
is that developers gain operational simplicity but lose control over the
execution environment, face cold-start latency, and must design within the
constraints of stateless, short-lived function execution.

## Structure and Components

### Component Diagram

```
+------------+  +------------+  +-----------+  +-----------+
| HTTP       |  | Message    |  | Schedule  |  | Storage   |
| Request    |  | Queue      |  | (Cron)    |  | Event     |
+-----+------+  +-----+------+  +-----+-----+  +-----+-----+
      |               |               |               |
      v               v               v               v
+-----+---------------+---------------+---------------+------+
|                    EVENT ROUTER / GATEWAY                   |
|  (Maps triggers to functions, handles authentication)      |
+--+--------+--------+--------+--------+--------+-----------+
   |        |        |        |        |        |
   v        v        v        v        v        v
+------+ +------+ +------+ +------+ +------+ +------+
|Func A| |Func B| |Func C| |Func D| |Func E| |Func F|
|Order | |Pay   | |Notify| |Report| |Resize| |Auth  |
|Create| |Process| |Send | |Gen   | |Image | |Verify|
+--+---+ +--+---+ +--+---+ +--+---+ +--+---+ +------+
   |        |        |        |        |
   v        v        v        v        v
+------+ +------+ +------+ +------+ +------+
|DB    | |Pay   | |Email | |Store | |Store |
|Table | |Gate  | |Svc   | |Bucket| |Bucket|
+------+ +------+ +------+ +------+ +------+

  Scaling: Each function scales independently (0 to N instances)
  Billing: Per invocation + execution duration
```

### Key Components

1. **Function**: A small, single-purpose unit of code that handles one
   type of event. Stateless, ephemeral, and independently deployable.

2. **Event Source / Trigger**: The stimulus that causes a function to
   execute: HTTP requests, message queue items, scheduled timers,
   database changes, file uploads, or custom events.

3. **API Gateway**: Routes HTTP requests to functions, handles
   authentication, rate limiting, and request transformation.

4. **Managed Services**: Backend services (databases, queues, storage,
   authentication) provided and operated by the cloud platform.

5. **Orchestrator**: A workflow engine that coordinates multi-step
   function executions, handling sequencing, branching, and error
   recovery.

## How It Works

```
  Event: "New file uploaded to storage"
       |
       v
  +----+--------+
  | Event       |  1. Platform detects the event
  | Source      |  2. Matches event to registered function
  +----+--------+
       |
       v
  +----+--------+
  | Platform    |  3. Provisions execution environment (cold start)
  | Runtime     |     OR reuses warm container (warm start)
  +----+--------+  4. Loads function code
       |           5. Injects event data as function parameter
       v
  +----+--------+
  | Function    |  6. Executes business logic
  | (User Code) |  7. Reads/writes to managed services
  +----+--------+  8. Returns result
       |
       v
  +----+--------+
  | Platform    |  9. Routes result to caller or next step
  | Runtime     | 10. May keep container warm for next invocation
  +----+--------+ 11. Scales to zero if no more events
```

## Pseudocode Example

```
// === FUNCTION: Process Order ===

FUNCTION HandleOrderCreated(event: OrderEvent, context: ExecutionContext) -> Response:
    LOG "Processing order " + event.orderId + " (request: " + context.requestId + ")"

    // Validate the event
    IF event.orderId IS NULL OR Length(event.items) = 0 THEN
        RETURN Response.BadRequest("Invalid order event")
    END IF

    // Idempotency check (critical for serverless -- events may be delivered twice)
    SET existing = Database.Get("processed_orders", event.orderId)
    IF existing IS NOT NULL THEN
        LOG "Order " + event.orderId + " already processed, skipping"
        RETURN Response.OK({status: "already_processed"})
    END IF

    // Calculate total
    SET total = 0
    FOR EACH item IN event.items DO
        SET total = total + (item.price * item.quantity)
    END FOR

    // Save order to managed database
    SET order = {
        id: event.orderId,
        customerId: event.customerId,
        items: event.items,
        total: total,
        status: "PENDING",
        createdAt: Now(),
        processedAt: Now()
    }
    Database.Put("orders", order)

    // Mark as processed for idempotency
    Database.Put("processed_orders", {id: event.orderId, processedAt: Now()})

    // Trigger downstream functions via events
    EventBus.Publish("order.validated", {
        orderId: event.orderId,
        customerId: event.customerId,
        total: total
    })

    RETURN Response.OK({orderId: event.orderId, status: "PENDING"})
END FUNCTION

// === FUNCTION: Process Payment ===

FUNCTION HandleOrderValidated(event: ValidatedOrderEvent, context: ExecutionContext) -> Response:
    LOG "Processing payment for order " + event.orderId

    // Idempotency
    SET existingPayment = Database.Get("payments", event.orderId)
    IF existingPayment IS NOT NULL THEN
        RETURN Response.OK({status: "already_processed"})
    END IF

    // Call external payment service
    SET paymentResult = PaymentService.Charge({
        customerId: event.customerId,
        amount: event.total,
        currency: "USD",
        reference: event.orderId
    })

    SET payment = {
        orderId: event.orderId,
        transactionId: paymentResult.transactionId,
        status: paymentResult.status,
        processedAt: Now()
    }
    Database.Put("payments", payment)

    IF paymentResult.status = "SUCCESS" THEN
        EventBus.Publish("payment.completed", {
            orderId: event.orderId,
            transactionId: paymentResult.transactionId
        })
    ELSE
        EventBus.Publish("payment.failed", {
            orderId: event.orderId,
            reason: paymentResult.failureReason
        })
    END IF

    RETURN Response.OK(payment)
END FUNCTION

// === FUNCTION: HTTP API Endpoint ===

FUNCTION HandleGetOrder(request: HttpRequest, context: ExecutionContext) -> HttpResponse:
    SET orderId = request.pathParams["orderId"]

    IF orderId IS NULL THEN
        RETURN HttpResponse(400, {error: "Order ID required"})
    END IF

    SET order = Database.Get("orders", orderId)
    IF order IS NULL THEN
        RETURN HttpResponse(404, {error: "Order not found"})
    END IF

    SET payment = Database.Get("payments", orderId)
    SET response = {
        id: order.id,
        status: order.status,
        total: order.total,
        items: order.items,
        paymentStatus: IF payment IS NOT NULL THEN payment.status ELSE "PENDING"
    }

    RETURN HttpResponse(200, response)
END FUNCTION

// === FUNCTION: Scheduled Cleanup ===

FUNCTION HandleDailyCleanup(event: ScheduleEvent, context: ExecutionContext):
    LOG "Running daily cleanup at " + Now()

    // Remove expired temporary data
    SET cutoff = Now() - Days(30)
    SET expiredRecords = Database.Query("processed_orders",
        "processedAt < :cutoff", {cutoff: cutoff})

    SET deletedCount = 0
    FOR EACH record IN expiredRecords DO
        Database.Delete("processed_orders", record.id)
        SET deletedCount = deletedCount + 1
    END FOR

    LOG "Cleaned up " + deletedCount + " expired records"
END FUNCTION

// === ORCHESTRATION: Multi-Step Workflow ===

WORKFLOW OrderFulfillmentWorkflow:
    INPUT: orderEvent

    STEP 1: validateOrder
        FUNCTION: HandleOrderCreated
        INPUT: orderEvent
        ON SUCCESS: GOTO processPayment
        ON FAILURE: GOTO handleError

    STEP 2: processPayment
        FUNCTION: HandleOrderValidated
        INPUT: {orderId: validateOrder.output.orderId}
        TIMEOUT: 30 seconds
        RETRY: 3 times with exponential backoff
        ON SUCCESS: GOTO sendNotification
        ON FAILURE: GOTO handlePaymentFailure

    STEP 3: sendNotification
        FUNCTION: SendOrderConfirmation
        INPUT: {orderId: processPayment.output.orderId}
        ON SUCCESS: END
        ON FAILURE: LOG warning, END  // Non-critical step

    ERROR HANDLER: handleError
        FUNCTION: LogAndAlert
        INPUT: {error: currentStep.error, orderId: orderEvent.orderId}

    ERROR HANDLER: handlePaymentFailure
        FUNCTION: CancelOrder
        INPUT: {orderId: orderEvent.orderId, reason: "Payment failed"}
END WORKFLOW
```

## When to Use

- **Event-driven workloads** triggered by uploads, messages, or API calls
  with unpredictable timing and volume.
- **Sporadic or bursty traffic** where paying for idle servers is wasteful.
- **Rapid prototyping** where eliminating infrastructure setup accelerates
  time to market.
- **Microservice decomposition** where individual functions represent the
  smallest possible service boundary.
- **Scheduled jobs** (reports, cleanup, data sync) that run periodically
  and do not need a persistent server.

## When NOT to Use

- **Long-running processes** that exceed function execution time limits
  (typically 5-15 minutes).
- **Stateful applications** that require in-memory state between requests.
- **Low-latency requirements** where cold-start delays are unacceptable.
- **High-throughput constant workloads** where reserved capacity is cheaper
  than per-invocation pricing.
- **Complex local development** where replicating the cloud environment
  is difficult and slows development cycles.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| E-commerce | Order processing pipeline | Event-driven, bursty during sales, zero idle cost |
| IoT | Sensor data ingestion | Unpredictable volume, event-triggered processing |
| Media | Image/video thumbnail generation | Triggered by upload, scales to zero between uploads |
| Finance | Transaction notification | Event-driven, low average volume with occasional spikes |
| DevOps | Automated deployment pipeline | Triggered by code commits, runs infrequently |

## Trade-offs

### Advantages

- **No server management**: Zero operational overhead for infrastructure.
- **Automatic scaling**: Scales to zero and to peak automatically.
- **Pay per use**: Billed only for actual execution time.
- **Fast deployment**: Deploy individual functions independently.
- **Built-in high availability**: Platform handles redundancy.

### Disadvantages

- **Cold starts**: First invocation after idle period has higher latency.
- **Execution limits**: Time, memory, and payload size constraints.
- **Vendor lock-in**: Functions often depend on provider-specific services.
- **Debugging difficulty**: Distributed, ephemeral execution is hard to trace.
- **Statelessness constraint**: No in-memory state between invocations.

## Comparison with Related Patterns

| Dimension | Serverless | Microservices | Monolith |
|-----------|-----------|--------------|----------|
| Deployment unit | Single function | Service | Entire application |
| Scaling granularity | Per function | Per service | Entire application |
| State management | External only | Per service | In-process |
| Operations burden | Minimal | High | Low-Medium |
| Cold start risk | Yes | No (always running) | No |
| Cost model | Per invocation | Per server-hour | Per server-hour |

## Evolution and Variations

- **Function-as-a-Service (FaaS)**: The core serverless model -- individual
  functions triggered by events.
- **Backend-as-a-Service (BaaS)**: Managed services (authentication,
  databases, storage) consumed directly from client applications.
- **Serverless Containers**: Container-based execution with serverless
  scaling (scale to zero, per-request billing) but without function
  size constraints.
- **Edge Functions**: Serverless functions deployed to edge locations
  near users for minimal latency.

## Key Takeaways

1. Design for idempotency. Functions may be invoked multiple times for
   the same event. Every function must produce the same result regardless
   of how many times it is called with the same input.
2. Cold starts are real. For latency-sensitive paths, use provisioned
   concurrency or keep functions warm.
3. Embrace managed services. Serverless works best when the entire stack
   (database, queue, storage) is managed, not just the compute.
4. Functions should be small and focused. One function, one responsibility,
   one trigger type.
5. Monitor costs carefully. Per-invocation pricing is cheap at low volume
   but can become expensive at scale if not optimized.
