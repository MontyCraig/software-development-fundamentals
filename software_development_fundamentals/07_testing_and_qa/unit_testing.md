# Unit Testing

## Overview

A unit test verifies the behavior of the smallest testable piece of software
in isolation from the rest of the system. The "unit" is typically a single
function or module. Unit tests are fast, deterministic, and form the broad
base of the testing pyramid.

---

## The Testing Pyramid

```
           /\
          /  \
         / E2E\         Few, slow, expensive
        /------\
       /Integr- \       Moderate number
      / ation    \
     /------------\
    /  Unit Tests  \    Many, fast, cheap
   /________________\

  Unit tests:        70-80% of all tests
  Integration tests: 15-20% of all tests
  End-to-end tests:  5-10% of all tests
```

Unit tests should vastly outnumber other test types. They run in milliseconds,
catch most regressions, and provide precise failure localization.

---

## Test Anatomy: Arrange-Act-Assert (AAA)

Every unit test follows a three-phase structure:

```
FUNCTION test_calculate_discount_for_premium_customer():

    // ARRANGE: Set up preconditions and inputs
    SET customer = NEW Customer(type: "premium", yearsActive: 5)
    SET order = NEW Order(total: 100.00)

    // ACT: Execute the behavior under test
    SET discount = CALL calculateDiscount(customer, order)

    // ASSERT: Verify the expected outcome
    ASSERT discount = 10.00

END FUNCTION
```

```
+----------+--------------------------------------------+
| Phase    | Purpose                                    |
+----------+--------------------------------------------+
| Arrange  | Create objects, set up state, prepare data |
| Act      | Call the function or method under test     |
| Assert   | Verify the result matches expectations     |
+----------+--------------------------------------------+
```

### Naming Conventions

Test names should describe the scenario and expected outcome:

```
Pattern: test_[unit]_[scenario]_[expectedResult]

Examples:
  test_calculateTax_withZeroAmount_returnsZero
  test_validateEmail_withMissingAtSymbol_returnsFalse
  test_sortList_withEmptyList_returnsEmptyList
  test_withdraw_withInsufficientFunds_raisesError
```

---

## Test Doubles

Test doubles replace real dependencies to isolate the unit under test.

### Types of Test Doubles

```
+--------+-------------------------------------------------------+
| Type   | Description                                           |
+--------+-------------------------------------------------------+
| Dummy  | Passed around but never used. Fills a parameter.      |
| Stub   | Returns predetermined answers to calls.               |
| Spy    | Records how it was called (args, count, order).       |
| Mock   | Pre-programmed with expectations about calls.         |
| Fake   | Working implementation but simplified (e.g., in-memory|
|        | database instead of real database).                    |
+--------+-------------------------------------------------------+
```

### Dummy Example

```
FUNCTION test_createUser_assigns_id():
    SET dummyLogger = NEW DummyLogger()   // Never actually used
    SET service = NEW UserService(dummyLogger)
    SET user = CALL service.create("Alice")
    ASSERT user.id IS NOT NULL
END FUNCTION
```

### Stub Example

```
FUNCTION test_getWeather_returns_forecast():
    // Stub always returns sunny weather
    SET weatherStub = NEW Stub()
    CALL weatherStub.when("getTemperature").thenReturn(72)

    SET display = NEW WeatherDisplay(weatherStub)
    SET result = CALL display.showCurrent()

    ASSERT result CONTAINS "72"
END FUNCTION
```

### Spy Example

```
FUNCTION test_checkout_sends_confirmation_email():
    SET emailSpy = NEW Spy()
    SET checkout = NEW CheckoutService(emailSpy)

    CALL checkout.complete(order)

    ASSERT emailSpy.wasCalledWith("send", order.customerEmail)
    ASSERT emailSpy.callCount("send") = 1
END FUNCTION
```

### Mock Example

```
FUNCTION test_processPayment_calls_gateway_with_correct_amount():
    SET gatewayMock = NEW Mock()
    CALL gatewayMock.expect("charge").withArgs(49.99).once()

    SET processor = NEW PaymentProcessor(gatewayMock)
    CALL processor.process(orderOf(49.99))

    CALL gatewayMock.verify()    // Fails if expectations not met
END FUNCTION
```

### Fake Example

```
MODULE InMemoryUserRepository IMPLEMENTS UserRepository:
    SET storage: Map<Text, User> = NEW EmptyMap()

    FUNCTION save(user: User) -> Void:
        SET storage[user.id] = user
    END FUNCTION

    FUNCTION findById(id: Text) -> User:
        RETURN storage[id]
    END FUNCTION
END MODULE

FUNCTION test_user_persistence():
    SET repo = NEW InMemoryUserRepository()    // Fake, not real database
    SET user = NEW User(id: "1", name: "Alice")
    CALL repo.save(user)
    SET found = CALL repo.findById("1")
    ASSERT found.name = "Alice"
END FUNCTION
```

---

## Test Isolation

Each test must be completely independent of other tests.

```
Rules of Isolation:

  1. Tests run in any order and produce the same result
  2. Tests do not share mutable state
  3. Each test sets up its own preconditions (Arrange phase)
  4. Each test cleans up after itself if it modifies external state
  5. Tests do not depend on the output of other tests
```

```
BAD (shared state):
  SET sharedList = [1, 2, 3]

  FUNCTION test_addItem():
      CALL sharedList.add(4)
      ASSERT LENGTH(sharedList) = 4      // Depends on initial state

  FUNCTION test_removeItem():
      CALL sharedList.remove(1)
      ASSERT LENGTH(sharedList) = 2      // Fails if test_addItem ran first!

GOOD (isolated):
  FUNCTION test_addItem():
      SET list = [1, 2, 3]               // Fresh setup
      CALL list.add(4)
      ASSERT LENGTH(list) = 4

  FUNCTION test_removeItem():
      SET list = [1, 2, 3]               // Fresh setup
      CALL list.remove(1)
      ASSERT LENGTH(list) = 2
```

---

## Code Coverage

Code coverage measures how much of the codebase is exercised by tests.

### Coverage Types

```
+-----------+--------------------------------------------------+
| Type      | What It Measures                                 |
+-----------+--------------------------------------------------+
| Line      | Percentage of lines executed by tests            |
| Branch    | Percentage of branches (IF/ELSE paths) taken     |
| Path      | Percentage of unique execution paths exercised   |
| Condition | Each boolean sub-expression evaluated both ways  |
+-----------+--------------------------------------------------+
```

### Coverage Example

```
FUNCTION categorize(score: Number) -> Text:
    IF score >= 90 THEN              // Branch 1
        RETURN "Excellent"
    ELSE IF score >= 70 THEN         // Branch 2
        RETURN "Good"
    ELSE                              // Branch 3
        RETURN "Needs Improvement"
    END IF
END FUNCTION

Tests for 100% branch coverage:
  test_categorize_excellent:  score = 95  -> "Excellent"   (Branch 1)
  test_categorize_good:       score = 80  -> "Good"        (Branch 2)
  test_categorize_needs_work: score = 50  -> "Needs..."    (Branch 3)
```

### Coverage Targets

```
+-----------------------------------+
| Coverage Target Guidelines        |
+-----------------------------------+
| 80%+ line coverage   | Good       |
| 90%+ line coverage   | Excellent  |
| 100% line coverage   | Diminishing|
|                       | returns    |
| Focus on branch       | More       |
| coverage              | valuable   |
| 100% path coverage    | Usually    |
|                       | impractical|
+-----------------------------------+
```

Coverage is a useful metric but not a quality metric. 100% coverage does not
mean the code is bug-free -- it means every line was executed, not that every
behavior was verified.

---

## Property-Based Testing

Instead of testing specific examples, property-based testing generates random
inputs and verifies that invariant properties always hold.

```
Traditional test (example-based):
  ASSERT sort([3, 1, 2]) = [1, 2, 3]

Property-based test:
  FOR 1000 random lists DO
      SET original = GENERATE randomList()
      SET sorted = CALL sort(original)

      // Property 1: Output has same length as input
      ASSERT LENGTH(sorted) = LENGTH(original)

      // Property 2: Output is ordered
      FOR i FROM 0 TO LENGTH(sorted) - 2 DO
          ASSERT sorted[i] <= sorted[i + 1]
      END FOR

      // Property 3: Output contains same elements as input
      ASSERT ELEMENTS(sorted) = ELEMENTS(original)
  END FOR
```

### When to Use Property-Based Testing

```
+---------------------------------------------+
| Good candidates:                            |
+---------------------------------------------+
| - Sorting algorithms                        |
| - Serialization/deserialization roundtrips   |
| - Mathematical functions (commutativity,    |
|   associativity, identity)                  |
| - Encoding/decoding                         |
| - Data structure invariants                 |
+---------------------------------------------+
```

---

## Common Unit Testing Mistakes

```
+---+-----------------------------------------------+
| 1 | Testing implementation details instead of     |
|   | behavior (fragile tests)                      |
+---+-----------------------------------------------+
| 2 | Multiple assertions testing different things  |
|   | in one test (unclear failures)                |
+---+-----------------------------------------------+
| 3 | Tests that depend on execution order           |
+---+-----------------------------------------------+
| 4 | Testing trivial code (getters, setters)        |
+---+-----------------------------------------------+
| 5 | Over-mocking (testing the mocks, not the code)|
+---+-----------------------------------------------+
| 6 | Flaky tests (pass/fail non-deterministically)  |
+---+-----------------------------------------------+
| 7 | Slow unit tests (database, network, file I/O)  |
+---+-----------------------------------------------+
| 8 | No tests for edge cases (nulls, empty inputs, |
|   | boundary values)                              |
+---+-----------------------------------------------+
```

---

## Unit Testing Checklist

```
+----------------------------------------------------------+
| Unit Testing Checklist                                   |
+----------------------------------------------------------+
| [ ] Each test follows Arrange-Act-Assert structure       |
| [ ] Test names describe scenario and expected outcome    |
| [ ] Tests are isolated (no shared mutable state)         |
| [ ] Dependencies are replaced with appropriate doubles   |
| [ ] Edge cases and boundary values are covered           |
| [ ] Tests run fast (milliseconds, not seconds)           |
| [ ] Tests are deterministic (same result every time)     |
| [ ] Branch coverage is at least 80%                      |
| [ ] Tests verify behavior, not implementation details    |
+----------------------------------------------------------+
```
