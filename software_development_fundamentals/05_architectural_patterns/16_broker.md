# Broker Architecture

## Overview and Intent

The Broker pattern introduces an intermediary component -- the broker -- that
decouples service consumers from service providers. Consumers send requests to
the broker, which locates an appropriate provider, forwards the request, and
returns the response. Neither consumer nor provider needs to know the other's
location, protocol, or implementation details.

The intent is to provide location transparency and simplify distributed system
communication. In a system with many services, direct point-to-point connections
create an unmanageable web of dependencies. The broker centralizes connection
management, service discovery, load balancing, and protocol translation,
allowing services to be added, removed, or relocated without affecting consumers.

The broker pattern is the foundation of message-oriented middleware, API
gateways, service meshes, and request-routing architectures. It sits between
the simplicity of direct client-server connections and the complexity of fully
decentralized peer-to-peer networks.

## Structure and Components

### Component Diagram

```
+----------+  +----------+  +----------+
| Consumer |  | Consumer |  | Consumer |
|    A     |  |    B     |  |    C     |
+----+-----+  +----+-----+  +----+-----+
     |             |              |
     | request     | request      | request
     v             v              v
+----+-------------+--------------+-----+
|                BROKER                  |
|                                        |
|  +---------------+  +---------------+ |
|  | Service       |  | Request       | |
|  | Registry      |  | Router        | |
|  | (Discovery)   |  | (Load Balance)| |
|  +---------------+  +---------------+ |
|  +---------------+  +---------------+ |
|  | Protocol      |  | Health        | |
|  | Translator    |  | Monitor       | |
|  +---------------+  +---------------+ |
+---+--------+--------+--------+--------+
    |        |        |        |
    v        v        v        v
+------+ +------+ +------+ +------+
|Svc 1 | |Svc 2 | |Svc 3 | |Svc 4 |
|      | |      | |  (x2) | |      |
+------+ +------+ +------+ +------+
                   +------+
                   |Svc 3 |
                   | copy  |
                   +------+

  Svc 3 has two instances: broker load-balances between them
```

### Key Components

1. **Broker**: The central intermediary that receives all requests, looks
   up the target service, routes the request, and returns the response.

2. **Service Registry**: A directory maintained by the broker that maps
   service names to network locations, versions, and health status.

3. **Request Router**: Selects the best service instance based on load
   balancing strategy, health, and affinity rules.

4. **Protocol Translator**: Converts between different communication
   protocols when consumers and providers use different formats.

5. **Health Monitor**: Continuously checks registered services for
   availability, removing unhealthy instances from the routing pool.

## How It Works

```
  1. Service registers with broker on startup

  Service                    Broker
     |                          |
     |  REGISTER                |
     |  name: "payment-svc"    |
     |  address: "10.0.1.5"    |
     |------------------------->|
     |                          | (stores in registry)
     |  ACK                     |
     |<-------------------------|

  2. Consumer sends request through broker

  Consumer                   Broker                   Provider
     |                          |                         |
     |  REQUEST                 |                         |
     |  service: "payment-svc" |                         |
     |------------------------->|                         |
     |                          | 3. Lookup in registry  |
     |                          | 4. Select instance     |
     |                          | 5. Forward request     |
     |                          |------------------------>|
     |                          |                         | 6. Process
     |                          |  7. Response            |
     |                          |<------------------------|
     |  8. Response             |                         |
     |<-------------------------|                         |
```

## Pseudocode Example

```
// === SERVICE REGISTRY ===

STRUCTURE ServiceInstance:
    serviceId: String
    serviceName: String
    version: String
    address: String
    port: Integer
    healthEndpoint: String
    metadata: Map of String to String
    status: InstanceStatus   // HEALTHY, UNHEALTHY, DRAINING
    lastHealthCheck: DateTime
END STRUCTURE

FUNCTION ServiceRegistry.Register(instance: ServiceInstance) -> Boolean:
    IF NOT this.instances.HasKey(instance.serviceName) THEN
        SET this.instances[instance.serviceName] = NEW List
    END IF

    SET existing = this.FindInstance(instance.serviceName, instance.address, instance.port)
    IF existing IS NOT NULL THEN
        existing.version = instance.version
        existing.metadata = instance.metadata
        existing.status = HEALTHY
        RETURN TRUE
    END IF

    SET instance.status = HEALTHY
    SET instance.lastHealthCheck = Now()
    Append(this.instances[instance.serviceName], instance)
    LOG "Registered " + instance.serviceName + " at " + instance.address + ":" + instance.port
    RETURN TRUE
END FUNCTION

FUNCTION ServiceRegistry.GetHealthyInstances(serviceName: String) -> List of ServiceInstance:
    SET list = this.instances[serviceName]
    IF list IS NULL THEN RETURN EMPTY LIST END IF
    RETURN Filter(list, FUNCTION(inst): RETURN inst.status = HEALTHY END FUNCTION)
END FUNCTION

// === LOAD BALANCER ===

STRUCTURE LoadBalancer:
    strategy: String  // "round-robin", "least-connections", "weighted"
    counters: Map of String to Integer
END STRUCTURE

FUNCTION LoadBalancer.Select(instances: List of ServiceInstance) -> ServiceInstance:
    IF Length(instances) = 0 THEN RETURN NULL END IF
    IF Length(instances) = 1 THEN RETURN instances[0] END IF

    IF this.strategy = "round-robin" THEN
        SET key = instances[0].serviceName
        SET counter = this.counters.GetOrDefault(key, 0)
        SET selected = instances[counter MOD Length(instances)]
        SET this.counters[key] = counter + 1
        RETURN selected

    ELSE IF this.strategy = "least-connections" THEN
        RETURN MinBy(instances, FUNCTION(inst): RETURN inst.activeConnections END FUNCTION)

    ELSE IF this.strategy = "weighted" THEN
        SET totalWeight = Sum(instances, FUNCTION(i): RETURN ParseInteger(i.metadata["weight"]) END FUNCTION)
        SET random = RandomInteger(0, totalWeight)
        SET cumulative = 0
        FOR EACH inst IN instances DO
            SET cumulative = cumulative + ParseInteger(inst.metadata["weight"])
            IF random < cumulative THEN RETURN inst END IF
        END FOR
        RETURN instances[Length(instances) - 1]
    END IF
END FUNCTION

// === BROKER ===

FUNCTION Broker.HandleRequest(request: BrokerRequest) -> BrokerResponse:
    SET startTime = Now()

    // Find healthy instances
    SET instances = this.registry.GetHealthyInstances(request.serviceName)
    IF Length(instances) = 0 THEN
        RETURN BrokerResponse.ServiceUnavailable("No healthy instances of " + request.serviceName)
    END IF

    // Version filtering
    IF request.minVersion IS NOT NULL THEN
        SET instances = Filter(instances, FUNCTION(inst): RETURN inst.version >= request.minVersion END FUNCTION)
        IF Length(instances) = 0 THEN
            RETURN BrokerResponse.ServiceUnavailable(request.serviceName + " v" + request.minVersion + "+ not available")
        END IF
    END IF

    // Select instance
    SET selected = this.loadBalancer.Select(instances)

    // Translate protocol if needed
    SET forwardPayload = request.payload
    IF request.protocol != selected.metadata["protocol"] THEN
        SET forwardPayload = this.translator.Translate(
            request.payload, fromProtocol: request.protocol,
            toProtocol: selected.metadata["protocol"]
        )
    END IF

    // Forward with retry
    SET maxRetries = 2
    SET attempt = 0

    LOOP
        SET attempt = attempt + 1
        TRY
            SET response = NetworkClient.Send(
                address: selected.address, port: selected.port,
                payload: forwardPayload, timeout: 10000
            )
            Metrics.Record("broker.request", {
                service: request.serviceName, duration: Now() - startTime, status: "success"
            })
            RETURN BrokerResponse.OK(response)

        CATCH error AS TimeoutError
            LOG "Timeout forwarding to " + selected.address
            IF attempt >= maxRetries THEN
                selected.status = UNHEALTHY
                RETURN BrokerResponse.GatewayTimeout("Service did not respond")
            END IF
            SET instances = Remove(instances, selected)
            IF Length(instances) = 0 THEN
                RETURN BrokerResponse.ServiceUnavailable("All instances exhausted")
            END IF
            SET selected = this.loadBalancer.Select(instances)
        END TRY
    END LOOP
END FUNCTION

// === HEALTH CHECKER ===

FUNCTION HealthChecker.Run(registry: ServiceRegistry):
    LOOP FOREVER DO
        Sleep(Duration.Seconds(10))
        FOR EACH serviceName, instances IN registry.instances DO
            FOR EACH instance IN instances DO
                TRY
                    SET response = NetworkClient.Get(
                        url: instance.address + ":" + instance.port + instance.healthEndpoint,
                        timeout: 3000
                    )
                    SET instance.status = IF response.status = 200 THEN HEALTHY ELSE UNHEALTHY
                CATCH error
                    SET instance.status = UNHEALTHY
                    LOG "Health check failed for " + serviceName + " at " + instance.address
                END TRY
                SET instance.lastHealthCheck = Now()
            END FOR
        END FOR
    END LOOP
END FUNCTION
```

## When to Use

- **Systems with many services** where direct point-to-point connections
  would create unmanageable complexity.
- **Dynamic service environments** where services are frequently added,
  removed, or relocated (cloud, container orchestration).
- **Multi-protocol environments** where services speak different protocols
  and need translation.
- **Load distribution requirements** where requests must be balanced
  across multiple service instances.
- **Service versioning** where multiple versions of a service coexist
  and consumers need version-aware routing.

## When NOT to Use

- **Simple systems** with a small, fixed number of services where direct
  connections are manageable.
- **Ultra-low-latency systems** where the extra hop through the broker
  adds unacceptable delay.
- **Fully decentralized systems** where no central component should exist.
  Use peer-to-peer instead.
- **Systems where the broker itself becomes a bottleneck** and cannot be
  scaled adequately.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| API management | API gateway for mobile apps | Centralized routing, auth, rate limiting |
| Microservices | Service mesh sidecar proxy | Transparent service discovery and load balancing |
| Message systems | Message queue broker | Decouple producers and consumers |
| IoT | Device gateway | Protocol translation, device management |
| Enterprise | Integration middleware | Connect heterogeneous enterprise systems |

## Trade-offs

### Advantages

- **Location transparency**: Consumers do not need to know service addresses.
- **Load balancing**: Automatic distribution across service instances.
- **Protocol translation**: Consumers and providers can use different protocols.
- **Health management**: Unhealthy instances are automatically excluded.
- **Centralized monitoring**: All traffic flows through a single observation point.

### Disadvantages

- **Single point of failure**: Broker must be highly available.
- **Added latency**: Every request adds a network hop through the broker.
- **Throughput bottleneck**: Broker must handle all traffic.
- **Complexity**: Broker configuration and management add operational burden.
- **Coupling to broker**: Systems depend on broker availability.

## Comparison with Related Patterns

| Dimension | Broker | Client-Server | SOA (ESB) |
|-----------|--------|--------------|-----------|
| Intermediary | Yes (broker) | No (direct) | Yes (ESB) |
| Discovery | Broker registry | Known address | ESB registry |
| Protocol translation | Optional | No | Full transformation |
| Load balancing | Built-in | External | Built-in |
| Complexity | Medium | Low | High |

## Evolution and Variations

- **Message Broker**: Specializes in asynchronous message delivery with
  queuing, topics, and guaranteed delivery.
- **API Gateway**: Specializes in HTTP/REST routing with authentication,
  rate limiting, and request transformation.
- **Service Mesh Sidecar**: Deploys a broker proxy alongside each service,
  distributing the broker function and eliminating the central bottleneck.
- **Federated Broker**: Multiple broker instances share registry data,
  providing geographic distribution and fault tolerance.

## Key Takeaways

1. The broker trades a central bottleneck for simplified service
   communication. The bottleneck must be mitigated through clustering,
   replication, and careful capacity planning.
2. Health checking is not optional. Without it, the broker routes
   requests to dead services, causing cascading failures.
3. The broker should be "thin" -- route and translate, but avoid
   business logic. Business logic in the broker creates a distributed
   monolith.
4. Modern service meshes distribute the broker function to sidecars,
   eliminating the central bottleneck while preserving the benefits.
5. Broker and peer-to-peer are at opposite ends of the centralization
   spectrum. Choose based on your consistency, latency, and operational
   complexity requirements.
