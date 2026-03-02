# Infrastructure as Code

## Introduction

Infrastructure as Code (IaC) is the practice of managing and provisioning
computing infrastructure through machine-readable definition files rather than
manual processes or interactive configuration tools. Infrastructure definitions
are version-controlled, reviewed, tested, and deployed using the same workflows
as application code.

```
  ┌───────────────────────────────────────────────────────────┐
  │           INFRASTRUCTURE AS CODE WORKFLOW                  │
  │                                                            │
  │  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌────────┐│
  │  │  Define  │──▶│  Review  │──▶│   Test   │──▶│ Apply  ││
  │  │  (.iac)  │   │  (PR)    │   │ (validate)│  │ (deploy)││
  │  └──────────┘   └──────────┘   └──────────┘   └────────┘│
  │       │                                            │      │
  │       │              Version Control               │      │
  │       └────────────────────────────────────────────┘      │
  │                                                            │
  │  Benefits:                                                 │
  │  - Reproducible environments                               │
  │  - Auditable change history                                │
  │  - Peer-reviewed infrastructure changes                    │
  │  - Automated testing and validation                        │
  │  - Disaster recovery from source control                   │
  └───────────────────────────────────────────────────────────┘
```

---

## Declarative vs Imperative Approaches

### Declarative Approach

You describe the **desired end state**. The infrastructure tool determines
what changes are necessary to reach that state.

```
RESOURCE "compute_instance" "web_server"
  SET name = "web-prod-01"
  SET instance_type = "medium"
  SET cpu_count = 4
  SET memory_gb = 16
  SET disk_gb = 100
  SET operating_system = "linux-latest"

  NETWORK
    SET subnet = "production-subnet"
    SET public_ip = TRUE
    SET security_group = "web-tier"
  END NETWORK

  TAGS
    SET environment = "production"
    SET team = "platform"
    SET managed_by = "iac"
  END TAGS
END RESOURCE

RESOURCE "load_balancer" "web_lb"
  SET name = "web-prod-lb"
  SET algorithm = "round-robin"
  SET health_check_path = "/health"
  SET health_check_interval = 30 SECONDS
  SET targets = [web_server.address]
END RESOURCE
```

**Advantage:** Idempotent -- running the same definition multiple times
produces the same result. The tool handles the "how."

### Imperative Approach

You describe the **exact steps** to execute. You control the order and logic.

```
FUNCTION provision_web_server()
  SET server = CREATE_INSTANCE(
    name = "web-prod-01",
    type = "medium",
    cpu = 4,
    memory = 16,
    disk = 100
  )

  WAIT UNTIL server.state == "running"

  ASSIGN_PUBLIC_IP(server)
  ATTACH_SECURITY_GROUP(server, "web-tier")
  ADD_TO_SUBNET(server, "production-subnet")

  SET lb = GET_LOAD_BALANCER("web-prod-lb")
  IF lb IS NOT FOUND THEN
    SET lb = CREATE_LOAD_BALANCER(
      name = "web-prod-lb",
      algorithm = "round-robin"
    )
  END IF

  REGISTER_TARGET(lb, server)
  CONFIGURE_HEALTH_CHECK(lb, path = "/health", interval = 30)

  TAG(server, {environment: "production", team: "platform"})

  RETURN server
END FUNCTION
```

**Advantage:** Full control over execution order and conditional logic.
**Trade-off:** You must handle idempotency, error recovery, and state tracking
yourself.

| Aspect          | Declarative             | Imperative              |
|-----------------|-------------------------|-------------------------|
| Defines         | Desired state           | Steps to execute        |
| Idempotency     | Built-in                | Must be implemented     |
| Learning curve  | Domain-specific syntax  | General programming     |
| Flexibility     | Constrained by model    | Unlimited               |
| Drift detection | Automatic               | Must be built           |

---

## State Management and Drift Detection

Infrastructure tools maintain a **state file** that records the current known
state of all managed resources. This state is compared against the desired
configuration to compute a plan of changes.

```
  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
  │   Desired   │      │   Current   │      │   Actual    │
  │   Config    │      │    State    │      │   Infra     │
  │  (source)   │      │   (file)    │      │  (reality)  │
  └──────┬──────┘      └──────┬──────┘      └──────┬──────┘
         │                    │                     │
         └──────────┬─────────┘                     │
                    ▼                               │
              ┌───────────┐                         │
              │   DIFF    │  ◀──── Plan phase       │
              │  (plan)   │                         │
              └─────┬─────┘                         │
                    ▼                               │
              ┌───────────┐                         │
              │   APPLY   │  ──────────────────────▶│
              │ (execute) │        Update           │
              └───────────┘        reality          │
```

### Drift Detection Pseudocode

```
FUNCTION detect_drift(state_file, actual_infrastructure)
  SET expected = LOAD(state_file)
  SET actual = QUERY(actual_infrastructure)
  SET drift_report = EMPTY LIST

  FOR EACH resource IN expected.resources DO
    SET actual_resource = FIND(actual, resource.id)

    IF actual_resource IS NOT FOUND THEN
      APPEND drift_report WITH {
        resource: resource.id,
        type: "DELETED_EXTERNALLY",
        expected: resource.properties,
        actual: NONE
      }
    ELSE
      FOR EACH property IN resource.properties DO
        IF actual_resource[property.name] != property.value THEN
          APPEND drift_report WITH {
            resource: resource.id,
            type: "PROPERTY_CHANGED",
            property: property.name,
            expected: property.value,
            actual: actual_resource[property.name]
          }
        END IF
      END FOR
    END IF
  END FOR

  // Check for resources that exist but are not in state
  FOR EACH actual_resource IN actual DO
    IF NOT EXISTS IN expected WITH id = actual_resource.id THEN
      APPEND drift_report WITH {
        resource: actual_resource.id,
        type: "UNMANAGED_RESOURCE",
        expected: NONE,
        actual: actual_resource.properties
      }
    END IF
  END FOR

  RETURN drift_report
END FUNCTION
```

---

## Resource Dependency Graph

Resources often depend on each other. The infrastructure tool builds a directed
acyclic graph (DAG) of dependencies to determine the correct order of
operations.

```
                    ┌───────────────┐
                    │  Network      │
                    │  (VPC/VLAN)   │
                    └───────┬───────┘
                            │
                 ┌──────────┼──────────┐
                 │          │          │
          ┌──────▼──┐  ┌───▼─────┐  ┌▼────────────┐
          │ Subnet  │  │ Subnet  │  │ Security     │
          │ (public)│  │(private)│  │ Rules        │
          └────┬────┘  └────┬────┘  └──────┬───────┘
               │            │              │
          ┌────▼────┐  ┌────▼────┐         │
          │  Load   │  │Database │         │
          │Balancer │  │Instance │◀────────┘
          └────┬────┘  └─────────┘
               │
          ┌────▼────┐
          │  Web    │
          │ Servers │
          └─────────┘

  Execution order (topological sort):
  1. Network
  2. Subnet (public), Subnet (private), Security Rules  [parallel]
  3. Load Balancer, Database Instance                     [parallel]
  4. Web Servers
```

```
FUNCTION resolve_execution_order(resources)
  SET graph = BUILD_DEPENDENCY_GRAPH(resources)
  SET order = TOPOLOGICAL_SORT(graph)

  IF CYCLE_DETECTED(graph) THEN
    FAIL WITH "Circular dependency found: {cycle_path}"
  END IF

  // Group independent resources for parallel execution
  SET execution_waves = EMPTY LIST
  SET completed = EMPTY SET

  WHILE completed.size < order.size DO
    SET wave = EMPTY LIST
    FOR EACH resource IN order DO
      IF resource NOT IN completed THEN
        SET deps = GET_DEPENDENCIES(resource)
        IF ALL deps IN completed THEN
          APPEND wave WITH resource
        END IF
      END IF
    END FOR
    APPEND execution_waves WITH wave
    ADD wave TO completed
  END WHILE

  RETURN execution_waves
END FUNCTION
```

---

## Immutable vs Mutable Infrastructure

### Mutable Infrastructure

Existing servers are updated in place. Configuration management applies changes
to running systems.

```
  Server v1.0 ──[update]──▶ Server v1.1 ──[patch]──▶ Server v1.2
       │                         │                        │
       └─── Same machine, accumulating changes over time ─┘
```

**Risk:** Configuration drift, "snowflake" servers that are hard to reproduce.

### Immutable Infrastructure

Servers are never modified after creation. Updates are deployed by replacing
entire instances with new ones built from a fresh image.

```
  Server v1.0 ─────────▶ [destroy]
  Server v1.1 ─────────────────────▶ [create from image v1.1]
  Server v1.2 ─────────────────────────────────▶ [create from image v1.2]

  Each server is identical to its image -- no drift possible
```

**Advantage:** Reproducibility, no configuration drift, reliable rollback.
**Trade-off:** Requires robust image-building pipeline and faster provisioning.

---

## Environment Promotion Strategy

Code and infrastructure changes should be promoted through a chain of
environments with increasing production fidelity.

```
  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌────────────┐
  │   DEV   │───▶│   QA    │───▶│ STAGING │───▶│ PRODUCTION │
  │         │    │         │    │         │    │            │
  │ Rapid   │    │ Full    │    │ Prod-   │    │ Live       │
  │ iteration│   │ test    │    │ mirror  │    │ traffic    │
  │ No gates│    │ suites  │    │ Final   │    │ Monitored  │
  │         │    │         │    │ checks  │    │            │
  └─────────┘    └─────────┘    └─────────┘    └────────────┘
    Auto-deploy    Auto-deploy    Manual gate    Manual gate
    on commit      on QA pass     + smoke tests  + approval
```

**Principles:**
- The same artifact (image, package) is promoted across environments; never
  rebuild for a different environment.
- Environment-specific settings come from configuration, not from the artifact.
- Staging should mirror production as closely as possible in topology and data
  volume (with anonymized data).

---

## Module and Template Reuse

Infrastructure definitions should be modular. Common patterns are extracted
into reusable templates.

```
TEMPLATE "web_service"
  PARAMETER name          TYPE string   REQUIRED
  PARAMETER instance_type TYPE string   DEFAULT "small"
  PARAMETER replicas      TYPE integer  DEFAULT 2
  PARAMETER port          TYPE integer  DEFAULT 8080

  RESOURCE "compute_instance" "{name}-server"
    SET count = replicas
    SET instance_type = instance_type
    SET user_data = LOAD_SCRIPT("setup_web.sh")
  END RESOURCE

  RESOURCE "load_balancer" "{name}-lb"
    SET targets = ["{name}-server"]
    SET listener_port = port
    SET health_check_path = "/health"
  END RESOURCE

  OUTPUT endpoint = "{name}-lb.address:{port}"
END TEMPLATE

// Usage:
USE TEMPLATE "web_service"
  SET name = "api"
  SET instance_type = "large"
  SET replicas = 4
  SET port = 443
END USE

USE TEMPLATE "web_service"
  SET name = "frontend"
  SET replicas = 2
END USE
```

---

## Testing Infrastructure Code

Infrastructure code should be tested at multiple levels, just like application
code.

```
  ┌─────────────────────────────────────────────────────┐
  │          INFRASTRUCTURE TESTING PYRAMID              │
  │                                                      │
  │                   ┌──────────┐                       │
  │                   │ End-to-  │  Deploy to a real     │
  │                   │  End     │  ephemeral env and    │
  │                   │          │  validate behavior    │
  │                ┌──┴──────────┴──┐                    │
  │                │  Integration   │  Plan against a    │
  │                │                │  mock provider and  │
  │                │                │  verify the plan    │
  │             ┌──┴────────────────┴──┐                 │
  │             │      Unit / Lint     │  Validate       │
  │             │                      │  syntax, types, │
  │             │                      │  naming, policy │
  │             └──────────────────────┘                 │
  └─────────────────────────────────────────────────────┘
```

### Testing Pseudocode

```
FUNCTION test_infrastructure_syntax(definition_files)
  FOR EACH file IN definition_files DO
    SET result = VALIDATE_SYNTAX(file)
    IF result.errors.count > 0 THEN
      FAIL WITH "Syntax errors in {file}: {result.errors}"
    END IF
  END FOR
  RETURN PASS
END FUNCTION

FUNCTION test_infrastructure_policy(plan)
  // Enforce organizational policies on the plan
  FOR EACH resource IN plan.resources DO
    IF resource.tags["managed_by"] != "iac" THEN
      FAIL WITH "Resource {resource.id} missing 'managed_by' tag"
    END IF

    IF resource.type == "compute_instance" THEN
      IF resource.properties.public_ip == TRUE THEN
        IF resource.properties.security_group IS EMPTY THEN
          FAIL WITH "Public instance {resource.id} has no security group"
        END IF
      END IF
    END IF

    IF resource.properties.encryption != TRUE THEN
      IF resource.type IN ["storage_bucket", "database", "disk"] THEN
        FAIL WITH "Resource {resource.id} must have encryption enabled"
      END IF
    END IF
  END FOR
  RETURN PASS
END FUNCTION

FUNCTION test_infrastructure_integration(definition, test_environment)
  SET plan = GENERATE_PLAN(definition, target = test_environment)

  ASSERT plan.destroy_count == 0
    MESSAGE "Integration test should not destroy existing resources"

  APPLY plan TO test_environment
  WAIT UNTIL all_resources_healthy(test_environment)

  RUN connectivity_tests AGAINST test_environment
  RUN security_scan AGAINST test_environment

  DESTROY test_environment  // Clean up ephemeral env
  RETURN PASS
END FUNCTION
```

---

## Practice Problems

1. **Declarative Design:** Write a declarative infrastructure definition for a
   two-tier application: a pool of application servers behind a load balancer,
   connected to a database in a private subnet. Include all dependencies.

2. **Drift Remediation:** Your drift detection finds that a production server's
   security group was manually modified to open an additional port. Describe
   two approaches to remediate this: one that preserves the manual change and
   one that enforces the code-defined state. What are the risks of each?

3. **Dependency Resolution:** Given these resources and dependencies, draw the
   dependency graph and determine the execution waves for parallel deployment:
   - Network (no dependencies)
   - Subnet A (depends on Network)
   - Subnet B (depends on Network)
   - Database (depends on Subnet B, Security Rules)
   - Security Rules (depends on Network)
   - App Server (depends on Subnet A, Database)
   - Load Balancer (depends on App Server)

4. **Environment Parity:** Your staging environment uses a smaller instance
   type than production to reduce costs. A performance bug only manifests in
   production. Propose a strategy that balances cost savings with production
   fidelity for the staging environment.

5. **Testing Strategy:** Write pseudocode for a policy test that ensures all
   storage resources have encryption enabled, all compute instances have a
   maximum lifetime tag, and no resource is deployed to a region outside an
   approved list.

6. **Module Design:** Design a reusable template for a "message queue" that
   accepts parameters for queue name, message retention period, dead-letter
   configuration, and encryption setting. Show how two different services
   would instantiate this template with different parameters.
