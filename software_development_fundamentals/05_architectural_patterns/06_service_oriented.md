# Service-Oriented Architecture (SOA)

## Overview and Intent

Service-Oriented Architecture organizes an enterprise system as a collection
of coarse-grained, reusable services that communicate through a centralized
infrastructure called an Enterprise Service Bus (ESB). Each service represents
a significant business capability and exposes a formal contract that defines
its interface, behavior, and quality-of-service characteristics.

The intent of SOA is to enable enterprise-wide reuse of business capabilities.
Rather than reimplementing common functions (customer lookup, credit check,
address validation) in every application, SOA extracts them into shared services
that any application can consume. The ESB acts as a universal translator and
router, mediating between services that may use different protocols, data
formats, and communication patterns.

SOA emerged in the early 2000s as a response to the "application silo" problem
in large enterprises. While it has been partially superseded by microservices for
greenfield development, SOA principles remain relevant in enterprise integration
scenarios where heterogeneous systems must interoperate.

## Structure and Components

### Component Diagram

```
+----------+  +----------+  +----------+  +----------+
| App A    |  | App B    |  | App C    |  | App D    |
| (CRM)    |  | (ERP)    |  | (Web     |  | (Mobile) |
|          |  |          |  |  Portal) |  |          |
+----+-----+  +----+-----+  +----+-----+  +----+-----+
     |             |             |             |
     v             v             v             v
+----+-------------+-------------+-------------+-----+
|              ENTERPRISE SERVICE BUS (ESB)           |
|                                                      |
|  +----------+  +----------+  +----------+           |
|  | Protocol |  | Message  |  | Service  |           |
|  | Mediator |  | Transform|  | Router   |           |
|  +----------+  +----------+  +----------+           |
|  +----------+  +----------+  +----------+           |
|  | Security |  | Logging  |  | Retry /  |           |
|  | Gateway  |  | / Audit  |  | Circuit  |           |
|  +----------+  +----------+  +----------+           |
+--+------+-------+--------+-------+--------+--------+
   |      |       |        |       |        |
   v      v       v        v       v        v
+------+ +------+ +------+ +------+ +------+ +------+
|Cust. | |Order | |Inven.| |Notif.| |Pricg.| |Auth. |
|Svc   | |Svc   | |Svc   | |Svc   | |Svc   | |Svc   |
+------+ +------+ +------+ +------+ +------+ +------+
   |      |       |        |       |        |
   v      v       v        v       v        v
+------+ +------+ +------+ +------+ +------+ +------+
| DB   | | DB   | | DB   | |Queue | | DB   | |LDAP  |
+------+ +------+ +------+ +------+ +------+ +------+
```

### Key Components

1. **Enterprise Service Bus (ESB)**: Centralized middleware that handles
   routing, protocol translation, message transformation, security
   enforcement, and monitoring.

2. **Service Contract**: Formal specification of a service's interface,
   including operations, message formats, security requirements, and
   quality-of-service guarantees.

3. **Service Provider**: A component that implements a service contract
   and registers itself with the service registry.

4. **Service Consumer**: An application or service that discovers and
   invokes services through the ESB.

5. **Service Registry**: A directory of available services, their contracts,
   endpoints, and metadata. Enables discovery at design time and runtime.

## How It Works

```
  Consumer                ESB                  Provider
     |                     |                      |
     |  1. Discover service|                      |
     |--(query registry)-->|                      |
     |<--(contract + URL)--|                      |
     |                     |                      |
     |  2. Send request    |                      |
     |--(protocol A)------>|                      |
     |                     | 3. Validate contract |
     |                     | 4. Transform message |
     |                     |    (format A -> B)   |
     |                     | 5. Route to provider |
     |                     |--(protocol B)------->|
     |                     |                      |  6. Process
     |                     |  7. Response         |
     |                     |<--(protocol B)-------|
     |                     | 8. Transform back    |
     |  9. Receive         |                      |
     |<--(protocol A)------|                      |
```

Step-by-step:

1. A consumer queries the service registry to find a service that matches
   its need (e.g., "customer credit check").
2. The consumer sends a request to the ESB using its native protocol.
3. The ESB validates the request against the service contract.
4. The ESB transforms the message from the consumer's format to the
   provider's expected format.
5. The ESB routes the transformed message to the appropriate provider.
6. The provider processes the request and returns a response.
7. The ESB transforms the response back to the consumer's format.
8. The consumer receives the response in its native format.

## Pseudocode Example

```
// --- Service Contract Definition ---

STRUCTURE CustomerServiceContract:
    name: "CustomerService"
    version: "2.1"
    operations:
        - GetCustomer(customerId: String) -> CustomerData
        - SearchCustomers(criteria: SearchCriteria) -> List of CustomerData
        - UpdateCustomer(customerId: String, data: CustomerUpdate) -> Result
    qualityOfService:
        maxResponseTime: 2000  // milliseconds
        availability: 99.9    // percent
        throughput: 500        // requests per second
    securityPolicy:
        authentication: "token"
        encryption: "transport"
        authorization: "role-based"
END STRUCTURE

// --- Service Registry ---

FUNCTION ServiceRegistry.Register(contract: ServiceContract, endpoint: String):
    SET entry = NEW RegistryEntry
    SET entry.serviceName = contract.name
    SET entry.version = contract.version
    SET entry.contract = contract
    SET entry.endpoint = endpoint
    SET entry.registeredAt = Now()
    SET entry.healthStatus = "HEALTHY"

    RegistryStore.Save(entry)
    LOG "Registered " + contract.name + " v" + contract.version + " at " + endpoint
END FUNCTION

FUNCTION ServiceRegistry.Discover(serviceName: String, minVersion: String) -> RegistryEntry:
    SET entries = RegistryStore.FindByName(serviceName)
    SET compatible = Filter(entries, FUNCTION(e):
        RETURN e.healthStatus = "HEALTHY" AND e.version >= minVersion
    END FUNCTION)

    IF Length(compatible) = 0 THEN
        RAISE ServiceNotFound(serviceName + " >= v" + minVersion)
    END IF

    // Return the highest compatible version
    RETURN SortByVersionDescending(compatible)[0]
END FUNCTION

// --- Enterprise Service Bus ---

FUNCTION ESB.HandleRequest(request: ServiceRequest) -> ServiceResponse:
    // Step 1: Authenticate the consumer
    SET identity = SecurityGateway.Authenticate(request.credentials)
    IF identity IS NULL THEN
        RETURN ServiceResponse.Unauthorized("Authentication failed")
    END IF

    // Step 2: Discover the target service
    SET entry = ServiceRegistry.Discover(request.serviceName, request.minVersion)

    // Step 3: Authorize the consumer for this service
    IF NOT SecurityGateway.Authorize(identity, entry.contract) THEN
        RETURN ServiceResponse.Forbidden("Not authorized for " + request.serviceName)
    END IF

    // Step 4: Validate request against the contract
    SET validation = ContractValidator.Validate(request.payload, entry.contract, request.operation)
    IF NOT validation.isValid THEN
        RETURN ServiceResponse.BadRequest(validation.errors)
    END IF

    // Step 5: Transform the message
    SET transformedPayload = MessageTransformer.Transform(
        request.payload,
        sourceFormat: request.format,
        targetFormat: entry.contract.messageFormat
    )

    // Step 6: Route and invoke
    SET startTime = Now()
    TRY
        SET providerResponse = ServiceInvoker.Call(
            endpoint: entry.endpoint,
            operation: request.operation,
            payload: transformedPayload,
            timeout: entry.contract.qualityOfService.maxResponseTime
        )

        // Step 7: Transform response back
        SET transformedResponse = MessageTransformer.Transform(
            providerResponse.payload,
            sourceFormat: entry.contract.messageFormat,
            targetFormat: request.format
        )

        // Step 8: Audit logging
        AuditLog.Record({
            consumer: identity.name,
            service: request.serviceName,
            operation: request.operation,
            responseTime: Now() - startTime,
            status: "SUCCESS"
        })

        RETURN ServiceResponse.OK(transformedResponse)

    CATCH error AS TimeoutError
        AuditLog.Record({
            consumer: identity.name,
            service: request.serviceName,
            operation: request.operation,
            responseTime: Now() - startTime,
            status: "TIMEOUT"
        })
        RETURN ServiceResponse.Timeout("Service did not respond in time")

    CATCH error AS ServiceError
        AuditLog.Record({
            consumer: identity.name,
            service: request.serviceName,
            operation: request.operation,
            responseTime: Now() - startTime,
            status: "ERROR",
            errorDetail: error.message
        })
        RETURN ServiceResponse.ServerError(error.message)
    END TRY
END FUNCTION

// --- Message Transformer ---

FUNCTION MessageTransformer.Transform(payload: Any, sourceFormat: String, targetFormat: String) -> Any:
    IF sourceFormat = targetFormat THEN
        RETURN payload
    END IF

    SET transformKey = sourceFormat + "->" + targetFormat
    SET transformer = this.transformerRegistry[transformKey]

    IF transformer IS NULL THEN
        RAISE TransformationError("No transformer for " + transformKey)
    END IF

    RETURN transformer.Apply(payload)
END FUNCTION

// --- Service Provider Implementation ---

FUNCTION CustomerServiceProvider.GetCustomer(customerId: String) -> CustomerData:
    SET customer = CustomerDatabase.FindById(customerId)
    IF customer IS NULL THEN
        RAISE ServiceError("Customer not found: " + customerId)
    END IF

    SET data = NEW CustomerData
    SET data.id = customer.id
    SET data.name = customer.firstName + " " + customer.lastName
    SET data.email = customer.email
    SET data.segment = customer.marketingSegment
    SET data.creditRating = CreditService.GetRating(customer.id)
    RETURN data
END FUNCTION
```

## When to Use

- **Enterprise integration** where multiple heterogeneous applications must
  share business capabilities across organizational boundaries.
- **Large organizations with governance requirements** that need centralized
  control over service contracts, security, and compliance.
- **Legacy system modernization** where existing systems must be wrapped
  with service interfaces without rewriting them.
- **Cross-departmental reuse** where services like customer lookup, address
  validation, or credit checking are needed by many applications.
- **Regulated industries** where audit logging, contract enforcement, and
  centralized security are mandatory.

## When NOT to Use

- **Greenfield applications** where microservices provide better team
  autonomy and deployment independence.
- **Small organizations** where the ESB infrastructure cost and complexity
  are not justified.
- **Agile, fast-moving teams** where centralized governance slows down
  development and deployment cycles.
- **Simple integrations** between two or three systems where point-to-point
  connections are simpler and sufficient.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Banking | Core banking integration | Multiple applications sharing customer, account services |
| Government | Citizen services portal | Many agencies sharing identity, address, benefits services |
| Healthcare | Hospital information system | EMR, billing, pharmacy sharing patient data services |
| Insurance | Policy administration | Underwriting, claims, billing sharing policy services |
| Telecom | Customer care platform | CRM, billing, provisioning sharing subscriber services |

## Trade-offs

### Advantages

- **Enterprise-wide reuse**: Services are shared across applications.
- **Protocol independence**: ESB translates between different protocols.
- **Centralized governance**: Contracts, security, and monitoring in one place.
- **Legacy integration**: Existing systems can be wrapped as services.
- **Audit and compliance**: Centralized logging of all service interactions.

### Disadvantages

- **ESB as bottleneck**: All traffic flows through the ESB.
- **Complexity**: ESB configuration, contract management, and transformation
  rules create significant operational burden.
- **Vendor lock-in**: ESB products are often proprietary and expensive.
- **Slow change**: Centralized governance can slow innovation.
- **Distributed monolith risk**: Poor design creates coupling through the ESB.

## Comparison with Related Patterns

| Dimension | SOA | Microservices | Event-Driven |
|-----------|-----|--------------|--------------|
| Service granularity | Coarse (enterprise-level) | Fine (team-level) | Event-based (any size) |
| Communication | ESB (centralized) | Direct or mesh | Event channel |
| Governance | Centralized | Decentralized | Per-topic |
| Contract style | Formal, versioned | Lightweight | Event schemas |
| Typical org size | Large enterprise | Any | Any |

## Evolution and Variations

- **Traditional SOA**: Heavy ESB, formal WSDL contracts, centralized
  registry. The original enterprise pattern.
- **Lightweight SOA**: Uses REST-like APIs instead of formal contracts,
  with a simple API gateway instead of a full ESB.
- **Event-Driven SOA**: Combines SOA service contracts with event-driven
  asynchronous communication for better decoupling.
- **API-Led Connectivity**: Modern take on SOA that organizes services into
  system APIs, process APIs, and experience APIs.

## Key Takeaways

1. SOA solves the enterprise integration problem by making business
   capabilities available as reusable, contractually defined services.
2. The ESB is both the pattern's strength (centralized mediation) and
   its weakness (single bottleneck, complexity magnet).
3. Service contracts are the key governance mechanism. They define what a
   service does, how to call it, and what quality to expect.
4. SOA works best in large enterprises with many heterogeneous systems.
   For smaller organizations, microservices or simple API gateways are
   more practical.
5. The principles of SOA (service reuse, formal contracts, loose coupling)
   remain valuable even when the ESB-centric implementation is replaced
   with lighter-weight alternatives.
