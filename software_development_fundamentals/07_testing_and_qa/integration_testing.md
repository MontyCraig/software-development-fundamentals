# Integration Testing

## Overview

Integration testing verifies that independently developed modules work
correctly when combined. While unit tests validate individual components in
isolation, integration tests validate the interactions, interfaces, and data
flow between those components.

```
Unit Testing:        [Module A]    [Module B]    [Module C]
                     (isolated)    (isolated)    (isolated)

Integration Testing: [Module A] <---> [Module B] <---> [Module C]
                     (interactions verified)
```

---

## Integration Strategies

### 1. Top-Down Integration

Start from the highest-level module and progressively integrate lower-level
modules. Stubs replace modules that are not yet integrated.

```
  Step 1:        [UI Layer]
                 /    |    \
              [Stub] [Stub] [Stub]

  Step 2:        [UI Layer]
                 /    |    \
          [Auth]   [Stub]  [Stub]

  Step 3:        [UI Layer]
                 /    |    \
          [Auth]  [Orders] [Stub]

  Step 4:        [UI Layer]
                 /    |    \
          [Auth]  [Orders] [Reports]
                     |
                  [Database]
```

```
+--------------------------------------------------+
| Top-Down Pros                                    |
+--------------------------------------------------+
| + Major control flow tested early                |
| + Early demonstration of working system skeleton |
| + Detects interface issues at top levels first   |
+--------------------------------------------------+
| Top-Down Cons                                    |
+--------------------------------------------------+
| - Many stubs needed for lower modules            |
| - Lower-level modules tested late                |
| - Stub maintenance can be significant            |
+--------------------------------------------------+
```

### 2. Bottom-Up Integration

Start from the lowest-level modules and progressively integrate higher-level
modules. Drivers simulate calls from modules above.

```
  Step 1:     [Driver]     [Driver]     [Driver]
                 |            |            |
          [Database]    [FileSystem]   [Network]

  Step 2:     [Driver]         [Driver]
                 |                |
          [Data Access]    [File Manager]
                 |                |
          [Database]       [FileSystem]

  Step 3:        [Business Logic]
                 /              \
          [Data Access]    [File Manager]
                 |                |
          [Database]       [FileSystem]
```

```
+--------------------------------------------------+
| Bottom-Up Pros                                   |
+--------------------------------------------------+
| + No stubs needed (real modules from the start)  |
| + Lower-level, reusable modules tested first     |
| + Easier to observe test results                 |
+--------------------------------------------------+
| Bottom-Up Cons                                   |
+--------------------------------------------------+
| - Top-level flow not tested until late           |
| - No early working system to demonstrate         |
| - Drivers must simulate higher-level behavior    |
+--------------------------------------------------+
```

### 3. Sandwich (Hybrid) Integration

Combines top-down and bottom-up approaches. Top layers and bottom layers are
integrated simultaneously toward the middle.

```
  Top-Down:      [UI Layer]
                  /     \
           [Auth]       [Orders]     <-- Meeting in the middle
                           |
  Bottom-Up:         [Data Access]
                         |
                    [Database]
```

```
+--------------------------------------------------+
| Sandwich Pros                                    |
+--------------------------------------------------+
| + Both high-level flow and low-level modules     |
|   tested early                                   |
| + Faster than pure top-down or bottom-up         |
| + Fewer stubs and drivers needed                 |
+--------------------------------------------------+
| Sandwich Cons                                    |
+--------------------------------------------------+
| - More complex to plan and coordinate            |
| - Middle layer may be tested last                |
+--------------------------------------------------+
```

### 4. Big Bang Integration

All modules are integrated simultaneously and tested as a complete system.

```
  All at once:
    [UI] + [Auth] + [Orders] + [Data Access] + [Database]
         +---> Test everything together <---+
```

```
+--------------------------------------------------+
| Big Bang Pros                                    |
+--------------------------------------------------+
| + No stubs or drivers needed                     |
| + Simple -- just combine and test                |
+--------------------------------------------------+
| Big Bang Cons                                    |
+--------------------------------------------------+
| - Difficult to isolate the source of failures    |
| - Late discovery of interface defects            |
| - Debugging is extremely difficult               |
| - Only feasible for very small systems           |
+--------------------------------------------------+
```

### Strategy Comparison

```
+------------------+----------+----------+----------+----------+
| Criteria         | Top-Down | Bottom-Up| Sandwich | Big Bang |
+------------------+----------+----------+----------+----------+
| Stub/Driver need | High     | High     | Low      | None     |
| Early demo       | Yes      | No       | Partial  | No       |
| Fault isolation  | Good     | Good     | Good     | Poor     |
| Interface issues | Top-first| Bot-first| Both     | All late |
| Complexity       | Medium   | Medium   | High     | Low      |
| Risk             | Low      | Low      | Low      | High     |
+------------------+----------+----------+----------+----------+
```

---

## Contract Testing

Contract testing verifies that the interface (contract) between a producer
and a consumer is maintained. It does not test full behavior, only that the
agreed-upon data shapes and interactions are respected.

```
  [Consumer] ---expects---> Contract <---fulfills--- [Producer]

  Consumer Contract Test:
    "When I send GET /users/1, I expect:
     - Status 200
     - Body with fields: id (Number), name (Text), email (Text)"

  Producer Contract Test:
    "When endpoint GET /users/1 is called:
     - I return status 200
     - Body contains: id (Number), name (Text), email (Text)"
```

### Contract Test Example

```
FUNCTION test_user_api_contract():
    // Consumer side: verify expected shape
    SET response = CALL apiClient.getUser("1")

    ASSERT response.status = 200
    ASSERT response.body HAS FIELD "id" OF TYPE Number
    ASSERT response.body HAS FIELD "name" OF TYPE Text
    ASSERT response.body HAS FIELD "email" OF TYPE Text
END FUNCTION
```

Contract tests are especially valuable for microservices where teams develop
independently but must maintain compatible interfaces.

---

## Test Environments

Integration tests require environments that resemble production.

```
Environment Progression:

  [Local Dev] -> [CI/CD] -> [Staging] -> [Production]
       |            |            |
   Mocked deps  Real deps   Production-like
   Fast          Moderate    Slow but accurate
```

```
+-------------------+----------------------------------------------+
| Environment       | Characteristics                              |
+-------------------+----------------------------------------------+
| Local Development | Mocked external services, in-memory stores,  |
|                   | fast feedback loop                           |
+-------------------+----------------------------------------------+
| CI/CD Pipeline    | Containerized dependencies, automated,       |
|                   | runs on every commit                         |
+-------------------+----------------------------------------------+
| Staging           | Production-like configuration, real external |
|                   | services (test accounts), full data set      |
+-------------------+----------------------------------------------+
| Production        | Real system (smoke tests only, not full      |
|                   | integration suites)                          |
+-------------------+----------------------------------------------+
```

---

## Database Testing Patterns

### Pattern 1: Transaction Rollback

Run each test inside a transaction that is rolled back after the test.

```
FUNCTION test_create_order():
    CALL database.beginTransaction()

    SET order = CALL orderService.create(items)
    ASSERT order.id IS NOT NULL
    SET found = CALL orderService.findById(order.id)
    ASSERT found.items = items

    CALL database.rollback()    // Clean up -- no data persists
END FUNCTION
```

### Pattern 2: Test Database Reset

Reset the database to a known state before each test or test suite.

```
FUNCTION beforeEachTest():
    CALL database.truncateAllTables()
    CALL database.loadFixtures("test_data.fixture")
END FUNCTION
```

### Pattern 3: In-Memory Database

Use a lightweight, in-memory database for speed.

```
FUNCTION setupTestDatabase():
    SET db = NEW InMemoryDatabase()
    CALL db.applySchema("schema.definition")
    RETURN db
END FUNCTION
```

---

## API Testing Patterns

### Request/Response Verification

```
FUNCTION test_create_user_api():
    SET request = NEW Request(
        method: "POST",
        path: "/api/users",
        body: { name: "Alice", email: "alice@example.test" }
    )

    SET response = CALL apiServer.handle(request)

    ASSERT response.status = 201
    ASSERT response.body.name = "Alice"
    ASSERT response.body.id IS NOT NULL
    ASSERT response.headers["Location"] CONTAINS "/api/users/"
END FUNCTION
```

### Error Handling Verification

```
FUNCTION test_create_user_duplicate_email():
    // Arrange: user already exists
    CALL createUser(name: "Alice", email: "alice@example.test")

    // Act: attempt duplicate
    SET request = NEW Request(
        method: "POST",
        path: "/api/users",
        body: { name: "Bob", email: "alice@example.test" }
    )
    SET response = CALL apiServer.handle(request)

    // Assert: proper error response
    ASSERT response.status = 409
    ASSERT response.body.error CONTAINS "already exists"
END FUNCTION
```

---

## Test Data Management

```
+-------------------------------+--------------------------------------+
| Strategy                      | When to Use                          |
+-------------------------------+--------------------------------------+
| Fixtures (static data files)  | Stable, well-known test scenarios    |
| Factories (generated data)    | Dynamic scenarios, many variations   |
| Snapshots (production copies) | Realistic data, anonymized           |
| Builders (programmatic)       | Complex objects with many fields     |
+-------------------------------+--------------------------------------+
```

### Factory Pattern for Test Data

```
MODULE UserFactory:
    SET counter = 0

    FUNCTION create(overrides: Map) -> User:
        SET counter = counter + 1
        SET defaults = {
            id: "user-" + counter,
            name: "User " + counter,
            email: "user" + counter + "@example.test",
            active: true
        }
        RETURN NEW User(MERGE(defaults, overrides))
    END FUNCTION
END MODULE

// Usage:
SET user1 = CALL UserFactory.create({})                     // defaults
SET user2 = CALL UserFactory.create({ active: false })      // inactive user
SET user3 = CALL UserFactory.create({ name: "Admin" })      // custom name
```

---

## Integration Testing Checklist

```
+----------------------------------------------------------+
| Integration Testing Checklist                            |
+----------------------------------------------------------+
| [ ] Integration strategy chosen (top-down, bottom-up,   |
|     sandwich, or big bang)                               |
| [ ] Test environment matches production topology         |
| [ ] Database state is reset between test runs            |
| [ ] External service contracts are verified              |
| [ ] Error conditions and failure modes are tested        |
| [ ] Data flows correctly between all module boundaries   |
| [ ] API request/response contracts are validated         |
| [ ] Test data is managed through factories or fixtures   |
| [ ] Tests run in CI/CD pipeline on every commit          |
+----------------------------------------------------------+
```

Integration tests bridge the gap between fast unit tests and slow end-to-end
tests, catching interface mismatches and interaction bugs that unit tests
cannot detect.
