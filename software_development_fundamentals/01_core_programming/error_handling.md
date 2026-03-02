# Error Handling

## Overview

Error handling is the practice of anticipating, detecting, and responding to abnormal
conditions that arise during program execution. Errors are inevitable: networks fail,
users provide invalid input, files go missing, and resources run out. A program that
does not handle errors gracefully will crash, corrupt data, or produce silently wrong
results.

Effective error handling separates the "happy path" (normal execution) from the
"unhappy path" (error conditions) in a way that keeps code readable and robust. The
goal is not to prevent all errors -- that is impossible -- but to ensure that when
errors occur, the program behaves predictably: reporting the problem clearly, cleaning
up resources, and either recovering or failing safely.

Understanding the spectrum of error handling strategies -- from exceptions to result
types, from defensive programming to fail-fast design -- equips developers to choose
the right approach for each situation and build systems that are both reliable and
maintainable.

## Key Concepts

### Exception-Based Error Handling

Exceptions are objects that represent an error condition. When raised (thrown), they
interrupt normal execution and propagate up the call stack until caught by a handler.

#### Try / Catch / Finally

```
FUNCTION readConfigFile(path: String) -> Configuration:
    TRY
        SET file = OPEN_FILE(path)
        SET content = READ(file)
        SET config = PARSE_CONFIG(content)
        RETURN config
    CATCH FileNotFoundError AS e
        PRINT "Config file missing: " + e.message
        RETURN DEFAULT_CONFIGURATION()
    CATCH ParseError AS e
        PRINT "Config file corrupt: " + e.message
        RAISE ConfigurationError("Invalid config at " + path, cause: e)
    FINALLY
        IF file IS NOT NULL THEN
            CLOSE(file)      // always runs, even if an exception occurred
        END IF
    END TRY
END FUNCTION
```

**Key rules**:
- `TRY` wraps the code that might fail.
- `CATCH` handles specific error types. Order matters: catch more specific types first.
- `FINALLY` runs regardless of success or failure -- used for cleanup.

### Error Propagation

Errors propagate up the call stack until handled. If no handler is found, the program
terminates.

```
FUNCTION main():
    TRY
        SET data = fetchData("https://api.example.com/data")
        SET processed = processData(data)
        saveResults(processed)
    CATCH NetworkError AS e
        PRINT "Network problem: " + e.message
    CATCH ProcessingError AS e
        PRINT "Data processing failed: " + e.message
    CATCH StorageError AS e
        PRINT "Could not save results: " + e.message
    END TRY
END FUNCTION

FUNCTION fetchData(url: String) -> Data:
    SET response = HTTP_GET(url)
    IF response.status != 200 THEN
        RAISE NetworkError("HTTP " + TO_STRING(response.status))
    END IF
    RETURN response.body
END FUNCTION
```

```
CALL STACK PROPAGATION

+------------------+
| main()           |  <-- catches NetworkError here
+------------------+
| fetchData()      |  <-- raises NetworkError
+------------------+
| HTTP_GET()       |  <-- actual failure point
+------------------+

The error bubbles UP from HTTP_GET -> fetchData -> main
until a matching CATCH block is found.
```

### Error Codes vs Exceptions

Two fundamentally different approaches to signaling errors:

```
+----------------------------+----------------------------+
| ERROR CODES                | EXCEPTIONS                 |
+----------------------------+----------------------------+
| Function returns a status  | Error objects propagate    |
| code; caller MUST check.   | automatically up the stack.|
|                            |                            |
| + Explicit control flow    | + Cannot be ignored        |
| + No hidden control jumps  | + Separate error path from |
| - Easy to forget to check  |   normal path              |
| - Clutters return values   | - Hidden control flow      |
| - Error codes can be       | - Performance cost for     |
|   misinterpreted           |   stack unwinding          |
+----------------------------+----------------------------+
```

```
// Error code approach
FUNCTION divide(a: Float, b: Float) -> (Float, ErrorCode):
    IF b = 0 THEN
        RETURN (0.0, ERROR_DIVISION_BY_ZERO)
    END IF
    RETURN (a / b, SUCCESS)
END FUNCTION

SET (result, err) = divide(10.0, 0.0)
IF err != SUCCESS THEN
    PRINT "Division failed"
END IF

// Exception approach
FUNCTION divide(a: Float, b: Float) -> Float:
    IF b = 0 THEN
        RAISE DivisionByZeroError("Cannot divide by zero")
    END IF
    RETURN a / b
END FUNCTION
```

### Result and Option Types

Result and Option types make error handling explicit in the type system without using
exceptions. They force the caller to handle both success and failure cases.

#### Option / Maybe Type

Represents a value that may or may not exist.

```
DEFINE TYPE Option<T>:
    CASE Some(value: T)     // value is present
    CASE None               // value is absent
END TYPE

FUNCTION findUser(id: String) -> Option<User>:
    SET user = DATABASE_LOOKUP(id)
    IF user EXISTS THEN
        RETURN Some(user)
    END IF
    RETURN None
END FUNCTION

// Caller must handle both cases
SET result = findUser("user-42")
MATCH result:
    CASE Some(user):
        PRINT "Found: " + user.name
    CASE None:
        PRINT "User not found"
END MATCH
```

#### Result Type

Represents either a success value or an error.

```
DEFINE TYPE Result<T, E>:
    CASE Ok(value: T)       // operation succeeded
    CASE Err(error: E)      // operation failed
END TYPE

FUNCTION parseInteger(input: String) -> Result<Integer, String>:
    IF IS_NUMERIC(input) THEN
        RETURN Ok(TO_INTEGER(input))
    END IF
    RETURN Err("'" + input + "' is not a valid integer")
END FUNCTION

// Chaining results
FUNCTION parseAndDouble(input: String) -> Result<Integer, String>:
    SET parsed = parseInteger(input)
    MATCH parsed:
        CASE Ok(value):
            RETURN Ok(value * 2)
        CASE Err(message):
            RETURN Err(message)       // propagate the error
    END MATCH
END FUNCTION

SET result = parseAndDouble("21")     // Ok(42)
SET failed = parseAndDouble("abc")    // Err("'abc' is not a valid integer")
```

### Defensive Programming

Defensive programming validates inputs and assumptions early, before they can cause
deeper problems.

```
FUNCTION transferFunds(from: Account, to: Account, amount: Float) -> Result:
    // Validate all inputs upfront
    IF from IS NULL THEN
        RAISE ArgumentError("Source account cannot be null")
    END IF
    IF to IS NULL THEN
        RAISE ArgumentError("Destination account cannot be null")
    END IF
    IF amount <= 0 THEN
        RAISE ArgumentError("Transfer amount must be positive")
    END IF
    IF from.balance < amount THEN
        RAISE InsufficientFundsError("Balance: " + TO_STRING(from.balance))
    END IF
    IF from.id = to.id THEN
        RAISE ArgumentError("Cannot transfer to the same account")
    END IF

    // All preconditions verified -- safe to proceed
    from.withdraw(amount)
    to.deposit(amount)
    RETURN Success
END FUNCTION
```

### Fail-Fast vs Fail-Safe

```
+-------------------------------+-------------------------------+
| FAIL-FAST                     | FAIL-SAFE                     |
+-------------------------------+-------------------------------+
| Stop immediately when an      | Continue operating in a       |
| error is detected.            | degraded mode when errors     |
|                               | occur.                        |
| Best for:                     | Best for:                     |
| - Development/testing         | - Production systems          |
| - Data corruption prevention  | - User-facing applications    |
| - Configuration errors        | - Non-critical features       |
+-------------------------------+-------------------------------+
```

```
// Fail-fast: crash on invalid configuration
FUNCTION loadConfig(path: String) -> Configuration:
    SET config = READ_CONFIG(path)
    IF config.databaseUrl IS EMPTY THEN
        RAISE FatalError("Database URL is required -- cannot start")
    END IF
    RETURN config
END FUNCTION

// Fail-safe: degrade gracefully
FUNCTION getProfileImage(userId: String) -> Image:
    TRY
        RETURN FETCH_IMAGE(userId)
    CATCH NetworkError:
        RETURN DEFAULT_AVATAR          // show placeholder instead of crashing
    END TRY
END FUNCTION
```

### Logging Errors

Logging provides visibility into errors that occur in production.

```
FUNCTION processOrder(order: Order) -> Result:
    TRY
        LOG.info("Processing order " + order.id)
        validateOrder(order)
        chargePayment(order)
        shipOrder(order)
        LOG.info("Order " + order.id + " completed successfully")
        RETURN Success
    CATCH ValidationError AS e
        LOG.warn("Order " + order.id + " validation failed: " + e.message)
        RETURN Failure(e.message)
    CATCH PaymentError AS e
        LOG.error("Payment failed for order " + order.id + ": " + e.message)
        RETURN Failure("Payment processing error")
    CATCH Error AS e
        LOG.critical("Unexpected error processing order " + order.id, e)
        RAISE e    // re-raise unexpected errors
    END TRY
END FUNCTION
```

**Log levels**: DEBUG < INFO < WARN < ERROR < CRITICAL

## Error Hierarchy

```
                    +-------------------+
                    |      Error        |
                    +--------+----------+
                             |
              +--------------+--------------+
              |                             |
    +---------+----------+      +-----------+---------+
    | RecoverableError   |      | FatalError          |
    +--------------------+      +---------------------+
    | Can be handled     |      | Program must stop   |
    +--------+-----------+      +----------+----------+
             |                             |
    +--------+--------+          +---------+---------+
    |                 |          |                   |
+---+----+     +------+---+  +--+--------+   +------+------+
|Validate|     | Network  |  | OutOf     |   | Corrupted   |
|Error   |     | Error    |  | Memory    |   | State       |
+--------+     +----------+  +-----------+   +-------------+
  |               |
  |          +----+----+
  |          |         |
+---+--+ +---+---+ +---+---+
|Format| |Timeout| |Refused|
+------+ +-------+ +-------+
```

## Retry Patterns

Transient errors (network timeouts, temporary service unavailability) often succeed on
retry.

```
FUNCTION fetchWithRetry(url: String, maxRetries: Integer = 3) -> Data:
    SET attempt = 0
    SET lastError = NULL

    WHILE attempt < maxRetries DO
        TRY
            SET response = HTTP_GET(url)
            RETURN response.body           // success -- exit loop
        CATCH NetworkError AS e
            SET lastError = e
            SET attempt = attempt + 1
            LOG.warn("Attempt " + TO_STRING(attempt) + " failed: " + e.message)

            IF attempt < maxRetries THEN
                // Exponential backoff: 1s, 2s, 4s, ...
                SET delay = POWER(2, attempt - 1) * 1000
                SLEEP(delay)
            END IF
        END TRY
    END WHILE

    RAISE RetryExhaustedError(
        "Failed after " + TO_STRING(maxRetries) + " attempts",
        cause: lastError
    )
END FUNCTION
```

```
RETRY WITH EXPONENTIAL BACKOFF

Attempt 1 ----X (fail)
              |
         [wait 1s]
              |
Attempt 2 --------X (fail)
                  |
             [wait 2s]
                  |
Attempt 3 ------------v (success!)

OR

Attempt 3 ----------------X (fail)
                          |
                     [wait 4s]
                          |
         RAISE RetryExhaustedError
```

### Circuit Breaker Pattern

For systems where repeated retries to a failed service cause cascading problems:

```
DEFINE CLASS CircuitBreaker:
    PRIVATE state: Enum = CLOSED    // CLOSED, OPEN, HALF_OPEN
    PRIVATE failureCount: Integer = 0
    PRIVATE threshold: Integer = 5
    PRIVATE resetTimeout: Integer = 30000

    METHOD execute(operation: Function) -> Result:
        IF this.state = OPEN THEN
            IF TIME_SINCE_LAST_FAILURE() > this.resetTimeout THEN
                SET this.state = HALF_OPEN
            ELSE
                RETURN Err("Circuit is OPEN -- service unavailable")
            END IF
        END IF

        TRY
            SET result = operation()
            SET this.failureCount = 0
            SET this.state = CLOSED
            RETURN Ok(result)
        CATCH Error AS e
            SET this.failureCount = this.failureCount + 1
            IF this.failureCount >= this.threshold THEN
                SET this.state = OPEN
                LOG.error("Circuit breaker OPEN after " + TO_STRING(this.threshold) + " failures")
            END IF
            RETURN Err(e.message)
        END TRY
    END METHOD
END CLASS
```

## Common Pitfalls and Best Practices

### Pitfalls

1. **Swallowing exceptions**: Catching an error and doing nothing with it hides bugs.
   At minimum, log the error.

2. **Catching too broadly**: Catching all errors masks unexpected problems. Catch only
   the specific errors you know how to handle.

3. **Using exceptions for control flow**: Exceptions are for exceptional conditions, not
   regular branching logic. They are slower and harder to follow.

4. **Inconsistent error handling**: Mixing error codes and exceptions in the same codebase
   without a clear strategy confuses developers.

5. **Missing resource cleanup**: Failing to close files, connections, or locks in error
   paths leads to resource leaks. Always use `FINALLY` or equivalent.

6. **Exposing internal errors to users**: Stack traces and internal error messages
   should be logged, not displayed to end users.

### Best Practices

1. **Handle errors at the right level**: Catch errors where you have enough context to
   respond meaningfully.
2. **Fail fast on programmer errors**: Invalid arguments, violated preconditions, and
   broken invariants should crash immediately in development.
3. **Degrade gracefully for user errors**: Invalid input should produce helpful error
   messages, not crashes.
4. **Use typed errors**: Result/Option types or specific exception classes convey more
   information than generic error codes.
5. **Always clean up resources**: Use `FINALLY` blocks or resource management patterns
   to prevent leaks.
6. **Include context in error messages**: "File not found: /etc/app/config.yaml" is
   far more useful than "File not found".
7. **Log at appropriate levels**: Distinguish between warnings (recoverable) and errors
   (requiring attention).

## Real-World Applications

- **Web servers**: Return appropriate HTTP status codes (400, 404, 500) based on error
  type; never expose stack traces to clients.
- **Payment processing**: Use Result types to handle declined cards, network failures,
  and validation errors distinctly.
- **File processing**: Defensive validation of file formats before parsing prevents
  corruption and crashes.
- **Distributed systems**: Circuit breakers and retry patterns maintain system stability
  when dependent services fail.
- **User interfaces**: Fail-safe patterns show degraded content rather than error screens.

## Related Topics

- [Variables and Data Types](variables_and_data_types.md) -- Null safety and type-related errors
- [Control Flow](control_flow.md) -- Guard clauses as defensive programming
- [Functions and Scope](functions_and_scope.md) -- Error propagation across function calls
- [Functional Programming](functional_programming.md) -- Result/Option types from FP
- [Object-Oriented Programming](object_oriented_programming.md) -- Exception hierarchies use inheritance

## Practice Problems

1. **Exception hierarchy**: Design an error hierarchy for an e-commerce system with at
   least 6 specific error types organized under 2-3 categories.

2. **Result type pipeline**: Write a chain of three functions that each return a
   `Result<T, Error>`, propagating errors through the chain without using exceptions.

3. **Retry with jitter**: Implement `fetchWithRetry` that adds random jitter to the
   exponential backoff delay to prevent thundering herd problems.

4. **Defensive validation**: Write a `validateRegistrationForm` function that collects
   ALL validation errors (not just the first) and returns them together.

5. **Resource cleanup**: Write a function that opens a database connection, runs a query,
   and ensures the connection is closed even if the query fails.

6. **Circuit breaker test**: Trace through a sequence of 8 calls to a circuit breaker
   with a threshold of 3, where calls 2, 3, and 4 fail. Identify when the circuit opens
   and what happens to calls 5 through 8.

7. **Error code to exceptions**: Refactor a function that uses error codes and nested
   if-checks into one that uses exceptions with proper try/catch structure.
