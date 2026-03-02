# Functions and Scope

## Overview

Functions are named, reusable blocks of code that accept inputs (parameters), perform a
computation, and optionally return an output. They are the primary mechanism for
decomposing complex programs into manageable, testable, and reusable parts. Every
significant program is built from layers of functions calling other functions.

Scope defines the visibility and lifetime of variables within a program. A variable
declared inside a function is typically invisible outside it, preventing accidental
interference between unrelated parts of the code. Understanding scope rules is critical
for reasoning about where a variable can be accessed, when it is created, and when it is
destroyed.

Together, functions and scope form the backbone of structured programming. They enable
abstraction (hiding details behind a function name), modularity (composing programs from
independent pieces), and encapsulation (limiting variable access to where it is needed).

## Key Concepts

### Function Declaration

A function is defined by its name, parameters, return type, and body:

```
FUNCTION add(a: Integer, b: Integer) -> Integer:
    RETURN a + b
END FUNCTION

FUNCTION greet(name: String) -> VOID:
    PRINT "Hello, " + name + "!"
END FUNCTION
```

### Parameters: By Value vs By Reference

#### Pass by Value

The function receives a copy. Changes inside the function do not affect the caller.

```
FUNCTION increment(x: Integer) -> Integer:
    SET x = x + 1
    RETURN x
END FUNCTION

SET original = 5
SET result = increment(original)
// result = 6, original = 5 (unchanged)
```

#### Pass by Reference

The function receives a reference to the original. Changes are visible to the caller.

```
FUNCTION appendItem(list: REFERENCE List<String>, item: String) -> VOID:
    APPEND list, item
END FUNCTION

SET names = ["Alice"]
appendItem(names, "Bob")
// names = ["Alice", "Bob"] (modified in place)
```

### Default Parameters

```
FUNCTION connect(host: String, port: Integer = 8080, secure: Boolean = TRUE) -> Connection:
    // port defaults to 8080, secure defaults to TRUE if not provided
    RETURN CREATE_CONNECTION(host, port, secure)
END FUNCTION

SET c1 = connect("example.com")             // port=8080, secure=TRUE
SET c2 = connect("example.com", 3000)        // port=3000, secure=TRUE
SET c3 = connect("example.com", 443, FALSE)  // port=443, secure=FALSE
```

### Return Values

Functions may return a single value, multiple values (via tuples or records), or nothing.

```
// Single return value
FUNCTION square(n: Integer) -> Integer:
    RETURN n * n
END FUNCTION

// Multiple return values
FUNCTION divideWithRemainder(a: Integer, b: Integer) -> (Integer, Integer):
    SET quotient = a / b
    SET remainder = a MOD b
    RETURN (quotient, remainder)
END FUNCTION

SET (q, r) = divideWithRemainder(17, 5)   // q=3, r=2

// No return value (side effect only)
FUNCTION logMessage(msg: String) -> VOID:
    WRITE_TO_FILE("log.txt", TIMESTAMP() + ": " + msg)
END FUNCTION
```

### Scope Chain

Variables are resolved by walking up the chain of enclosing scopes.

```
SET globalVar = "I am global"

FUNCTION outer() -> VOID:
    SET outerVar = "I am in outer"

    FUNCTION inner() -> VOID:
        SET innerVar = "I am in inner"
        PRINT globalVar    // found in global scope
        PRINT outerVar     // found in enclosing (outer) scope
        PRINT innerVar     // found in local scope
    END FUNCTION

    inner()
    PRINT innerVar         // ERROR: innerVar is not visible here
END FUNCTION
```

## Visual Representation

### Scope Chain Diagram

```
+--------------------------------------------------+
| GLOBAL SCOPE                                     |
|   globalVar = "I am global"                      |
|                                                  |
|  +--------------------------------------------+  |
|  | FUNCTION outer() SCOPE                     |  |
|  |   outerVar = "I am in outer"               |  |
|  |                                            |  |
|  |  +--------------------------------------+  |  |
|  |  | FUNCTION inner() SCOPE               |  |  |
|  |  |   innerVar = "I am in inner"         |  |  |
|  |  |                                      |  |  |
|  |  |   Lookup order:                      |  |  |
|  |  |   1. inner (local)                   |  |  |
|  |  |   2. outer (enclosing)               |  |  |
|  |  |   3. global                          |  |  |
|  |  +--------------------------------------+  |  |
|  +--------------------------------------------+  |
+--------------------------------------------------+
```

### Call Stack Visualization

```
FUNCTION main():
    SET x = factorial(4)
END FUNCTION

FUNCTION factorial(n: Integer) -> Integer:
    IF n <= 1 THEN RETURN 1 END IF
    RETURN n * factorial(n - 1)
END FUNCTION


Call stack grows downward:

+---------------------+
| main()              |  x = factorial(4)
+---------------------+
| factorial(4)        |  4 * factorial(3)
+---------------------+
| factorial(3)        |  3 * factorial(2)
+---------------------+
| factorial(2)        |  2 * factorial(1)
+---------------------+
| factorial(1)        |  RETURN 1          <-- base case
+---------------------+

Stack unwinds upward:

factorial(1) returns 1
factorial(2) returns 2 * 1 = 2
factorial(3) returns 3 * 2 = 6
factorial(4) returns 4 * 6 = 24
main() receives 24
```

### Lexical Scope

Lexical (static) scope means that a function's access to variables is determined by where
the function is defined in the source code, not where it is called.

```
SET multiplier = 10

FUNCTION makeMultiplier() -> Function:
    SET multiplier = 2                    // local to makeMultiplier
    FUNCTION multiply(x: Integer) -> Integer:
        RETURN x * multiplier             // uses multiplier=2 (lexical scope)
    END FUNCTION
    RETURN multiply
END FUNCTION

SET fn = makeMultiplier()
PRINT fn(5)    // 10, because multiplier=2 in the scope where fn was defined
```

### Closures

A closure is a function that captures and retains access to variables from its enclosing
scope, even after that scope has finished executing.

```
FUNCTION makeCounter() -> Function:
    SET count = 0

    FUNCTION increment() -> Integer:
        SET count = count + 1
        RETURN count
    END FUNCTION

    RETURN increment
END FUNCTION

SET counter = makeCounter()
PRINT counter()    // 1
PRINT counter()    // 2
PRINT counter()    // 3
// count is preserved in the closure between calls
```

### First-Class Functions

When functions are first-class, they can be assigned to variables, passed as arguments,
and returned from other functions.

```
// Assign to a variable
SET operation = add

// Pass as an argument
FUNCTION applyTwice(fn: Function, value: Integer) -> Integer:
    RETURN fn(fn(value))
END FUNCTION

FUNCTION doubleIt(x: Integer) -> Integer:
    RETURN x * 2
END FUNCTION

SET result = applyTwice(doubleIt, 3)   // doubleIt(doubleIt(3)) = doubleIt(6) = 12
```

### Higher-Order Functions

A higher-order function either takes a function as a parameter or returns a function.

```
FUNCTION filter(items: List<Integer>, predicate: Function) -> List<Integer>:
    SET result = []
    FOR EACH item IN items DO
        IF predicate(item) THEN
            APPEND result, item
        END IF
    END FOR
    RETURN result
END FUNCTION

FUNCTION isPositive(n: Integer) -> Boolean:
    RETURN n > 0
END FUNCTION

SET data = [-3, 7, -1, 4, 0, 9]
SET positives = filter(data, isPositive)   // [7, 4, 9]
```

### Recursion Introduction

Recursion occurs when a function calls itself. Every recursive function needs a base case
to prevent infinite recursion.

```
FUNCTION fibonacci(n: Integer) -> Integer:
    // Base cases
    IF n = 0 THEN RETURN 0 END IF
    IF n = 1 THEN RETURN 1 END IF

    // Recursive case
    RETURN fibonacci(n - 1) + fibonacci(n - 2)
END FUNCTION

// Tail-recursive version (optimizable by some runtimes)
FUNCTION fibonacciTail(n: Integer, a: Integer = 0, b: Integer = 1) -> Integer:
    IF n = 0 THEN RETURN a END IF
    RETURN fibonacciTail(n - 1, b, a + b)
END FUNCTION
```

## Common Pitfalls and Best Practices

### Pitfalls

1. **Unintended variable shadowing**: Declaring a local variable with the same name as
   an outer variable hides the outer one, leading to confusing bugs.

2. **Excessive global state**: Relying on global variables creates hidden dependencies
   between functions and makes testing difficult.

3. **Stack overflow**: Recursion without a proper base case or with too many levels
   exhausts the call stack.

4. **Side effects in unexpected places**: Functions that modify external state (files,
   globals, passed-by-reference parameters) without clear documentation cause surprises.

5. **Closure over loop variables**: Capturing a loop variable in a closure may bind
   to the final value rather than the value at each iteration.

### Best Practices

1. **Single responsibility**: Each function should do one thing well.
2. **Descriptive names**: `calculateMonthlyPayment()` not `calc()`.
3. **Limit parameters**: More than 3-4 parameters suggests the function needs refactoring
   or a parameter object.
4. **Prefer pure functions**: Functions that depend only on their inputs and produce no
   side effects are easier to test and reason about.
5. **Keep functions short**: If a function exceeds 20-30 lines, consider splitting it.
6. **Document side effects**: If a function modifies state, make it explicit in the name
   or documentation.

## Real-World Applications

- **Event handlers**: Closures capture state for callback-driven systems.
- **Configuration builders**: Higher-order functions create parameterized behavior.
- **Middleware chains**: Functions compose to form request-processing pipelines.
- **Recursive data processing**: Tree traversal, directory walking, nested structure
  parsing all use recursion.
- **API design**: Clean function signatures form the public interface of libraries.

## Related Topics

- [Variables and Data Types](variables_and_data_types.md) -- Parameter types and return types
- [Control Flow](control_flow.md) -- Branching and looping inside function bodies
- [Functional Programming](functional_programming.md) -- Functions as the core building block
- [Object-Oriented Programming](object_oriented_programming.md) -- Methods as functions bound to objects
- [Error Handling](error_handling.md) -- Exception propagation across function calls

## Practice Problems

1. **Power function**: Write a recursive function `POWER(base, exponent)` that computes
   `base` raised to `exponent` without using a built-in power operator.

2. **Closure counter**: Create a `makeCounter(start)` function that returns two closures:
   `increment()` and `getCount()`, sharing the same counter state.

3. **Higher-order transformation**: Write a `map` function that takes a list and a
   transformation function, returning a new list with the function applied to each element.

4. **Scope detective**: Given a nested function structure with shadowed variables, trace
   through the code and predict the output of each print statement.

5. **Callback pattern**: Write a function `fetchData(url, onSuccess, onError)` that
   simulates an asynchronous operation and calls the appropriate callback.

6. **Tail recursion**: Convert the naive recursive `sumList(list)` function into a
   tail-recursive version with an accumulator parameter.
