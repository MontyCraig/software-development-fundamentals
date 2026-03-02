# Containerization

## Introduction

Containerization is a lightweight virtualization technique that packages an
application together with its dependencies into an isolated, reproducible unit.
Unlike full virtual machines, containers share the host operating system kernel,
making them faster to start, smaller in size, and more efficient with resources.

```
  ┌─────────────────────────────────────────────────────────────┐
  │                CONTAINER vs VIRTUAL MACHINE                  │
  │                                                              │
  │  CONTAINERS                      VIRTUAL MACHINES            │
  │  ┌───────┐ ┌───────┐ ┌───────┐  ┌───────┐ ┌───────┐       │
  │  │ App A │ │ App B │ │ App C │  │ App A │ │ App B │       │
  │  ├───────┤ ├───────┤ ├───────┤  ├───────┤ ├───────┤       │
  │  │ Libs  │ │ Libs  │ │ Libs  │  │ Libs  │ │ Libs  │       │
  │  └───┬───┘ └───┬───┘ └───┬───┘  ├───────┤ ├───────┤       │
  │      │         │         │      │Guest  │ │Guest  │       │
  │  ┌───▼─────────▼─────────▼───┐  │  OS   │ │  OS   │       │
  │  │   Container Runtime       │  └───┬───┘ └───┬───┘       │
  │  ├───────────────────────────┤  ┌───▼─────────▼───┐       │
  │  │      Host OS Kernel       │  │   Hypervisor    │       │
  │  ├───────────────────────────┤  ├─────────────────┤       │
  │  │      Hardware             │  │   Host OS       │       │
  │  └───────────────────────────┘  ├─────────────────┤       │
  │                                 │   Hardware      │       │
  │  Startup: seconds               └─────────────────┘       │
  │  Size: megabytes               Startup: minutes            │
  │  Isolation: process-level      Size: gigabytes             │
  │                                Isolation: hardware-level    │
  └─────────────────────────────────────────────────────────────┘
```

---

## Image Layering

A container image is built from a series of read-only layers. Each instruction
in the build definition creates a new layer on top of the previous one. At
runtime, a thin writable layer is added on top.

```
  ┌─────────────────────────────────┐
  │  Writable Layer (runtime)       │  ◀── Container-specific changes
  ├─────────────────────────────────┤
  │  Layer 5: Application code      │  ◀── COPY application files
  ├─────────────────────────────────┤
  │  Layer 4: Dependencies          │  ◀── INSTALL packages
  ├─────────────────────────────────┤
  │  Layer 3: Configuration         │  ◀── SET environment variables
  ├─────────────────────────────────┤
  │  Layer 2: System packages       │  ◀── INSTALL system libraries
  ├─────────────────────────────────┤
  │  Layer 1: Base operating system │  ◀── FROM base_image
  └─────────────────────────────────┘
```

**Key principles of layering:**

- Layers are cached and shared across images. If two images use the same base,
  that base layer is stored only once on disk.
- Place instructions that change infrequently (base OS, system packages) early
  in the definition and frequently changing content (application code) late.
  This maximizes cache reuse.
- Each layer adds to the image size. Remove temporary files within the same
  layer that creates them.

### Image Build Pseudocode

```
FUNCTION build_image(definition_file, tag)
  SET layers = PARSE(definition_file)
  SET image_id = EMPTY

  FOR EACH layer IN layers DO
    SET cache_key = HASH(layer.instruction, layer.content, image_id)

    IF CACHE_EXISTS(cache_key) THEN
      SET image_id = CACHE_GET(cache_key)
      LOG "Using cached layer: {cache_key}"
    ELSE
      SET image_id = EXECUTE_LAYER(layer, parent = image_id)
      CACHE_STORE(cache_key, image_id)
      LOG "Built new layer: {cache_key}"
    END IF
  END FOR

  TAG image_id AS tag
  RETURN image_id
END FUNCTION
```

---

## Container Lifecycle: Build, Ship, Run

```
  ┌──────────┐         ┌──────────┐         ┌──────────┐
  │          │         │          │         │          │
  │  BUILD   │────────▶│   SHIP   │────────▶│   RUN    │
  │          │         │          │         │          │
  │ Define   │         │ Push to  │         │ Pull and │
  │ layers,  │         │ registry,│         │ start on │
  │ compile, │         │ version, │         │ target   │
  │ package  │         │ sign     │         │ host     │
  │          │         │          │         │          │
  └──────────┘         └──────────┘         └──────────┘
       │                    │                    │
       ▼                    ▼                    ▼
  Image created       Image stored         Container running
  locally             centrally            in production
```

### Lifecycle Pseudocode

```
FUNCTION container_lifecycle(definition, registry, target_host)
  // BUILD
  SET image = build_image(definition, tag = "app:latest")
  RUN security_scan ON image
  IF vulnerabilities WITH severity >= "high" THEN
    FAIL WITH "High severity vulnerabilities found"
  END IF

  // SHIP
  SIGN image WITH deployment_key
  PUSH image TO registry

  // RUN
  ON target_host DO
    PULL image FROM registry
    VERIFY_SIGNATURE(image)
    SET container = CREATE_CONTAINER(image, config)
    START container
    WAIT UNTIL health_check(container) == HEALTHY
    ROUTE traffic TO container
  END ON
END FUNCTION
```

---

## Orchestration Concepts

When you run many containers across multiple hosts, you need an orchestration
layer to manage scheduling, scaling, service discovery, and self-healing.

```
  ┌─────────────────────────────────────────────────────────┐
  │                    ORCHESTRATOR                          │
  │                                                         │
  │  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  │
  │  │  Scheduler  │  │   Scaler     │  │  Service     │  │
  │  │             │  │              │  │  Discovery   │  │
  │  │ Place work  │  │ Add/remove   │  │  Find other  │  │
  │  │ on nodes    │  │ instances    │  │  services    │  │
  │  └──────┬──────┘  └──────┬───────┘  └──────┬───────┘  │
  │         │                │                  │          │
  │  ┌──────▼──────────────▼────────────────▼──────────┐  │
  │  │               CLUSTER OF NODES                    │  │
  │  │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐         │  │
  │  │  │Node 1│  │Node 2│  │Node 3│  │Node 4│         │  │
  │  │  │[A][B]│  │[A][C]│  │[B][C]│  │[A][B]│         │  │
  │  │  └──────┘  └──────┘  └──────┘  └──────┘         │  │
  │  └──────────────────────────────────────────────────┘  │
  └─────────────────────────────────────────────────────────┘
```

### Scheduling

The scheduler decides which node should run a given container based on:
- Available resources (CPU, memory, disk)
- Affinity and anti-affinity rules (colocate or separate certain workloads)
- Node health and capacity
- Data locality requirements

### Scaling

```
FUNCTION auto_scale(service, metrics)
  SET current_count = COUNT_INSTANCES(service)
  SET cpu_avg = AVERAGE(metrics.cpu_usage, window = 5 MINUTES)
  SET request_rate = SUM(metrics.requests, window = 1 MINUTE)

  IF cpu_avg > 80 PERCENT OR request_rate > threshold_high THEN
    SET desired = MINIMUM(current_count * 2, max_instances)
    SCALE service TO desired INSTANCES
    LOG "Scaled up {service} to {desired}"
  ELSE IF cpu_avg < 20 PERCENT AND request_rate < threshold_low THEN
    SET desired = MAXIMUM(current_count / 2, min_instances)
    SCALE service TO desired INSTANCES
    LOG "Scaled down {service} to {desired}"
  END IF
END FUNCTION
```

### Service Discovery

Services need to find each other without hardcoded addresses. The orchestrator
maintains a service registry that maps service names to network endpoints.

```
  ┌──────────┐     ┌────────────────────┐     ┌──────────┐
  │ Service  │────▶│  Service Registry  │◀────│ Service  │
  │    A     │     │                    │     │    B     │
  │          │     │  "api" → 10.0.1.5  │     │          │
  │ Lookup:  │     │  "db"  → 10.0.1.8  │     │ Register │
  │ "db"     │     │  "cache"→ 10.0.1.3 │     │ "api"    │
  └──────────┘     └────────────────────┘     └──────────┘
```

---

## Service Mesh Pattern

A service mesh is an infrastructure layer that handles service-to-service
communication. Each container gets a sidecar proxy that intercepts all network
traffic, providing observability, security, and traffic management without
changing application code.

```
  ┌──────────────────────┐       ┌──────────────────────┐
  │      Service A       │       │      Service B       │
  │  ┌────────┐          │       │          ┌────────┐  │
  │  │  App   │          │       │          │  App   │  │
  │  └───┬────┘          │       │          └───┬────┘  │
  │      │               │       │              │       │
  │  ┌───▼────┐          │       │          ┌───▼────┐  │
  │  │Sidecar │◀────────mTLS──────────────▶│Sidecar │  │
  │  │ Proxy  │          │       │          │ Proxy  │  │
  │  └────────┘          │       │          └────────┘  │
  └──────────────────────┘       └──────────────────────┘
           │                               │
           └───────────┬───────────────────┘
                       ▼
              ┌─────────────────┐
              │  Control Plane  │
              │  - Routing      │
              │  - Policy       │
              │  - Telemetry    │
              └─────────────────┘
```

**Capabilities provided by a service mesh:**
- Mutual TLS encryption between services
- Automatic retries and circuit breaking
- Traffic splitting for canary deployments
- Distributed tracing injection
- Rate limiting and access control

---

## Health Checks and Readiness Probes

Orchestrators use two types of probes to manage container lifecycle:

- **Liveness probe:** Is the container still running? If not, restart it.
- **Readiness probe:** Is the container ready to accept traffic? If not,
  remove it from the load balancer.

```
FUNCTION liveness_check(container)
  SET response = HTTP_GET(container.address, path = "/health/live")
  IF response.status == 200 THEN
    RETURN HEALTHY
  ELSE
    SET container.failure_count = container.failure_count + 1
    IF container.failure_count >= 3 THEN
      RESTART container
      SET container.failure_count = 0
    END IF
    RETURN UNHEALTHY
  END IF
END FUNCTION

FUNCTION readiness_check(container)
  SET response = HTTP_GET(container.address, path = "/health/ready")

  IF response.status == 200 THEN
    IF container NOT IN load_balancer THEN
      ADD container TO load_balancer
      LOG "Container {container.id} is now receiving traffic"
    END IF
    RETURN READY
  ELSE
    IF container IN load_balancer THEN
      REMOVE container FROM load_balancer
      LOG "Container {container.id} removed from traffic"
    END IF
    RETURN NOT_READY
  END IF
END FUNCTION

FUNCTION startup_probe(container, max_wait)
  SET elapsed = 0
  SET interval = 5 SECONDS

  WHILE elapsed < max_wait DO
    SET response = HTTP_GET(container.address, path = "/health/started")
    IF response.status == 200 THEN
      RETURN STARTED
    END IF
    WAIT interval
    SET elapsed = elapsed + interval
  END WHILE

  LOG "Container {container.id} failed to start within {max_wait}"
  KILL container
  RETURN FAILED
END FUNCTION
```

---

## Resource Limits and Quotas

Containers must declare and respect resource boundaries to prevent one workload
from starving others.

```
  ┌────────────────────────────────────────────────────┐
  │                   HOST NODE                         │
  │  Total CPU: 8 cores    Total Memory: 32 GB         │
  │                                                     │
  │  ┌─────────────┐  ┌─────────────┐  ┌────────────┐ │
  │  │ Container A │  │ Container B │  │Container C │ │
  │  │ CPU: 2 req  │  │ CPU: 1 req  │  │CPU: 0.5 req│ │
  │  │ CPU: 4 lim  │  │ CPU: 2 lim  │  │CPU: 1 lim  │ │
  │  │ Mem: 4G req │  │ Mem: 8G req │  │Mem: 2G req │ │
  │  │ Mem: 8G lim │  │ Mem: 12G lim│  │Mem: 4G lim │ │
  │  └─────────────┘  └─────────────┘  └────────────┘ │
  │                                                     │
  │  Allocated: 3.5/8 CPU (req), 14/32 GB mem (req)   │
  │  Reserved:  7/8 CPU (lim), 24/32 GB mem (lim)     │
  └────────────────────────────────────────────────────┘

  req = request (guaranteed minimum)
  lim = limit (maximum allowed)
```

```
FUNCTION enforce_resource_limits(container, limits)
  SET cpu_limit = limits.cpu_max
  SET memory_limit = limits.memory_max

  IF container.cpu_usage > cpu_limit THEN
    THROTTLE container CPU TO cpu_limit
    LOG "Container {container.id} CPU throttled"
  END IF

  IF container.memory_usage > memory_limit THEN
    KILL container WITH reason "out of memory"
    LOG "Container {container.id} killed: exceeded memory limit"
  END IF
END FUNCTION
```

---

## Container Networking

Containers communicate over virtual networks. The orchestrator manages network
namespaces, virtual bridges, and routing rules.

```
  ┌──────────────────────────────────────────────────────────┐
  │  HOST NETWORK STACK                                       │
  │                                                           │
  │  ┌──────────────────────────────────────────────────┐    │
  │  │           Virtual Bridge / Overlay Network        │    │
  │  └──────┬──────────┬──────────┬──────────┬──────────┘    │
  │         │          │          │          │               │
  │    ┌────▼───┐ ┌────▼───┐ ┌───▼────┐ ┌──▼─────┐        │
  │    │ veth0  │ │ veth1  │ │ veth2  │ │ veth3  │        │
  │    │10.0.0.2│ │10.0.0.3│ │10.0.0.4│ │10.0.0.5│        │
  │    ├────────┤ ├────────┤ ├────────┤ ├────────┤        │
  │    │ App A  │ │ App B  │ │ App C  │ │ App D  │        │
  │    └────────┘ └────────┘ └────────┘ └────────┘        │
  │                                                           │
  │  Port Mapping:                                            │
  │    Host:8080  ──▶  App A:3000                             │
  │    Host:8081  ──▶  App B:3000                             │
  └──────────────────────────────────────────────────────────┘
```

**Networking models:**

- **Bridge networking:** Containers on the same host share a virtual bridge.
  Simple but limited to single-host communication.
- **Overlay networking:** A virtual network spans multiple hosts. Containers
  on different machines communicate as if on the same LAN.
- **Host networking:** Container shares the host's network namespace. Maximum
  performance but no isolation.

### Network Policy Pseudocode

```
FUNCTION apply_network_policy(service, policy)
  FOR EACH rule IN policy.ingress DO
    IF rule.source NOT IN allowed_services THEN
      BLOCK traffic FROM rule.source TO service
    ELSE
      ALLOW traffic FROM rule.source TO service ON rule.ports
    END IF
  END FOR

  FOR EACH rule IN policy.egress DO
    IF rule.destination NOT IN allowed_endpoints THEN
      BLOCK traffic FROM service TO rule.destination
    ELSE
      ALLOW traffic FROM service TO rule.destination ON rule.ports
    END IF
  END FOR

  LOG "Network policy applied to {service}: {policy.name}"
END FUNCTION
```

---

## Practice Problems

1. **Layer Optimization:** You have a container image definition that installs
   system packages, copies application code, then installs application
   dependencies. Every code change rebuilds all layers. Reorder the steps to
   maximize layer cache hits and explain why.

2. **Scaling Design:** Write pseudocode for a scaling function that considers
   both CPU usage and memory pressure. It should scale up when either metric
   exceeds a threshold and scale down only when both are below their respective
   low thresholds for a sustained period.

3. **Service Discovery Failure:** Service A cannot reach Service B after a
   deployment. List the diagnostic steps you would take, considering DNS,
   service registry, network policy, and health check states.

4. **Resource Planning:** A host has 16 CPU cores and 64 GB memory. You need
   to run Service X (requires 2 CPU, 8 GB each, 4 instances) and Service Y
   (requires 1 CPU, 4 GB each, 6 instances). Can they fit? What happens if
   Service X instances each request their limit of 4 CPU simultaneously?

5. **Network Isolation:** Design a network policy (in pseudocode) for a
   three-tier architecture: a frontend service, a backend API, and a database.
   The frontend may only talk to the API. The API may talk to both the frontend
   (responses) and the database. The database accepts connections only from
   the API.

6. **Health Check Design:** A container runs a background worker that processes
   jobs from a queue. It has no HTTP endpoint. Design liveness and readiness
   probes that do not use HTTP, explaining what each probe checks and how the
   orchestrator should respond to failures.
