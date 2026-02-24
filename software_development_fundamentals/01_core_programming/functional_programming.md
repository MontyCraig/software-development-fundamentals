# Functional Programming

## Overview

Functional Programming (FP) is a paradigm that treats computation as the evaluation of
mathematical functions and avoids changing state and mutable data. Instead of telling the
computer how to perform a task step by step (imperative style), FP describes what the
result should be in terms of function transformations applied to data.

The core insight of functional programming is that programs become easier to reason about,
test, and parallelize when functions have no side effects and data is immutable. A
function that always returns the same output for the same input (a pure function) can
be understood in isolation, composed with other functions freely, and executed in any
order or concurrently without risk of interference.

Functional programming has moved from academic research into mainstream practice. Its
concepts -- pure functions, immutability, higher-order functions, and declarative data
transformations -- are now found in virtually every modern programming environment.
Understanding FP principles makes any developer more effective, even when working in
primarily imperative or object-oriented codebases.

## Key Concepts

### Pure Functions

A pure function has two properties:
1. **Deterministic**: Same inputs always produce the same output.
2. **No side effects**: It does not modify any external state.

```
// PURE: depends only on inputs, modifies nothing external
FUNCTION add(a: Integer, b: Integer) -> Integer:
    RETURN a + b
END FUNCTION

// PURE: no external dependencies
FUNCTION fullName(first: String, last: String) -> String:
    RETURN first + " " + last
END FUNCTION

// IMPURE: depends on external state
SET taxRate = 0.08
FUNCTION calculateTax(price: Float) -> Float:
    RETURN price * taxRate     // depends on external variable
END FUNCTION

// IMPURE: produces side effect
FUNCTION logAndAdd(a: Integer, b: Integer) -> Integer:
    PRINT "Adding " + TO_STRING(a) + " and " + TO_STRING(b)   // side effect
    RETURN a + b
END FUNCTION
```

### Referential Transparency

An expression is referentially transparent if it can be replaced with its value without
changing the program's behavior. Pure functions are always referentially transparent.

```
// Referentially transparent
SET result = add(3, 4) + add(3, 4)
// Can be replaced with:
SET result = 7 + 7
// Same behavior guaranteed

// NOT referentially transparent
SET result = readUserInput() + readUserInput()
// Cannot replace with a single value -- each call may return different input
```

### Immutability

In FP, data is never modified after creation. Instead, new data structures are produced
from existing ones.

```
// MUTABLE approach (imperative)
SET scores = [85, 92, 78]
APPEND scores, 95              // modifies the original list
REMOVE scores[0]               // modifies the original list

// IMMUTABLE approach (functional)
SET scores = [85, 92, 78]
SET withNewScore = CONCAT(scores, [95])        // new list: [85, 92, 78, 95]
SET withoutFirst = SLICE(scores, 1, LENGTH(scores))  // new list: [92, 78]
// original scores is unchanged: [85, 92, 78]
```

```
MUTABLE DATA                         IMMUTABLE DATA
+-----------+                        +-----------+
| scores    |                        | scores    |----> [85, 92, 78]  (preserved)
+-----------+                        +-----------+
| [85,92,78]| --> [85,92,78,95]      | withNew   |----> [85, 92, 78, 95]  (new)
|           | --> [92,78,95]         +-----------+
+-----------+                        | noFirst   |----> [92, 78]  (new)
  (mutated in place)                 +-----------+
                                       (all versions coexist)
```

### Higher-Order Functions

Higher-order functions take functions as arguments or return functions as results. They
are the primary tool for abstraction in FP.

```
// Takes a function as argument
FUNCTION applyToAll(items: List<Integer>, fn: Function) -> List<Integer>:
    SET result = []
    FOR EACH item IN items DO
        APPEND result, fn(item)
    END FOR
    RETURN result
END FUNCTION

FUNCTION double(x: Integer) -> Integer:
    RETURN x * 2
END FUNCTION

SET doubled = applyToAll([1, 2, 3, 4], double)   // [2, 4, 6, 8]

// Returns a function
FUNCTION multiplier(factor: Integer) -> Function:
    FUNCTION multiply(x: Integer) -> Integer:
        RETURN x * factor
    END FUNCTION
    RETURN multiply
END FUNCTION

SET triple = multiplier(3)
PRINT triple(5)    // 15
PRINT triple(10)   // 30
```

### Map, Filter, Reduce

These three higher-order functions form the core of functional data transformation.

#### Map: Transform Each Element

```
FUNCTION map(items: List<T>, transform: Function) -> List<U>:
    SET result = []
    FOR EACH item IN items DO
        APPEND result, transform(item)
    END FOR
    RETURN result
END FUNCTION

SET names = ["alice", "bob", "carol"]
SET uppercased = map(names, TO_UPPERCASE)     // ["ALICE", "BOB", "CAROL"]

SET numbers = [1, 2, 3, 4, 5]
SET squared = map(numbers, FUNCTION(x): RETURN x * x END)  // [1, 4, 9, 16, 25]
```

#### Filter: Select Elements

```
FUNCTION filter(items: List<T>, predicate: Function) -> List<T>:
    SET result = []
    FOR EACH item IN items DO
        IF predicate(item) THEN
            APPEND result, item
        END IF
    END FOR
    RETURN result
END FUNCTION

SET numbers = [1, 2, 3, 4, 5, 6, 7, 8]
SET evens = filter(numbers, FUNCTION(x): RETURN x MOD 2 = 0 END)  // [2, 4, 6, 8]
```

#### Reduce: Combine into Single Value

```
FUNCTION reduce(items: List<T>, combiner: Function, initial: U) -> U:
    SET accumulator = initial
    FOR EACH item IN items DO
        SET accumulator = combiner(accumulator, item)
    END FOR
    RETURN accumulator
END FUNCTION

SET numbers = [1, 2, 3, 4, 5]
SET sum = reduce(numbers, FUNCTION(acc, x): RETURN acc + x END, 0)     // 15
SET product = reduce(numbers, FUNCTION(acc, x): RETURN acc * x END, 1) // 120

// Building a string
SET words = ["Functional", "programming", "is", "powerful"]
SET sentence = reduce(words, FUNCTION(acc, w): RETURN acc + " " + w END, "")
// " Functional programming is powerful"
```

### Data Transformation Pipeline

```
// Problem: Find the total cost of all in-stock items priced over 10.00

SET inventory = [
    {name: "Widget",  price: 25.00, inStock: TRUE},
    {name: "Gadget",  price: 8.50,  inStock: TRUE},
    {name: "Gizmo",   price: 15.75, inStock: FALSE},
    {name: "Doohick", price: 42.00, inStock: TRUE},
    {name: "Thingam", price: 3.25,  inStock: TRUE}
]

SET totalCost = inventory
    |> filter(FUNCTION(item): RETURN item.inStock END)
    |> filter(FUNCTION(item): RETURN item.price > 10.00 END)
    |> map(FUNCTION(item): RETURN item.price END)
    |> reduce(FUNCTION(acc, price): RETURN acc + price END, 0)

// Result: 25.00 + 42.00 = 67.00
```

```
PIPELINE VISUALIZATION

 [Widget, Gadget, Gizmo, Doohick, Thingam]
                    |
            filter(inStock)
                    |
     [Widget, Gadget, Doohick, Thingam]
                    |
          filter(price > 10)
                    |
           [Widget, Doohick]
                    |
            map(get price)
                    |
            [25.00, 42.00]
                    |
            reduce(sum, 0)
                    |
               67.00
```

### Function Composition

Composition combines two or more functions into a new function where the output of one
becomes the input of the next.

```
FUNCTION compose(f: Function, g: Function) -> Function:
    FUNCTION composed(x: ANY) -> ANY:
        RETURN f(g(x))
    END FUNCTION
    RETURN composed
END FUNCTION

FUNCTION addOne(x: Integer) -> Integer:
    RETURN x + 1
END FUNCTION

FUNCTION doubleIt(x: Integer) -> Integer:
    RETURN x * 2
END FUNCTION

SET addOneThenDouble = compose(doubleIt, addOne)
PRINT addOneThenDouble(3)     // doubleIt(addOne(3)) = doubleIt(4) = 8

SET doubleThenAddOne = compose(addOne, doubleIt)
PRINT doubleThenAddOne(3)     // addOne(doubleIt(3)) = addOne(6) = 7
```

```
COMPOSITION DIAGRAM

Input ---> [g: addOne] ---> [f: doubleIt] ---> Output

  3   --->    4        --->      8        ---> 8
```

### Currying

Currying transforms a function that takes multiple arguments into a chain of functions,
each taking a single argument.

```
// Multi-argument function
FUNCTION addThree(a: Integer, b: Integer, c: Integer) -> Integer:
    RETURN a + b + c
END FUNCTION

// Curried version
FUNCTION curriedAdd(a: Integer) -> Function:
    RETURN FUNCTION(b: Integer) -> Function:
        RETURN FUNCTION(c: Integer) -> Integer:
            RETURN a + b + c
        END FUNCTION
    END FUNCTION
END FUNCTION

// Full application
SET result = curriedAdd(1)(2)(3)    // 6

// Partial application
SET addFive = curriedAdd(2)(3)      // a function that adds 5
PRINT addFive(10)                   // 15
PRINT addFive(20)                   // 25
```

### Monads (Simplified)

A monad is a design pattern that wraps values in a context and provides a way to chain
operations on those wrapped values. The simplest way to understand monads is through
practical examples.

```
// The Maybe/Option monad -- handles the absence of a value

DEFINE MONAD Maybe<T>:
    CASE Some(value: T)
    CASE None
END MONAD

FUNCTION safeDivide(a: Float, b: Float) -> Maybe<Float>:
    IF b = 0 THEN
        RETURN None
    END IF
    RETURN Some(a / b)
END FUNCTION

FUNCTION safeSquareRoot(x: Float) -> Maybe<Float>:
    IF x < 0 THEN
        RETURN None
    END IF
    RETURN Some(SQRT(x))
END FUNCTION

// Chaining with BIND (also called flatMap or >>=)
SET result = Some(16.0)
    |> BIND(FUNCTION(x): RETURN safeDivide(x, 4.0) END)    // Some(4.0)
    |> BIND(FUNCTION(x): RETURN safeSquareRoot(x) END)      // Some(2.0)

SET failed = Some(16.0)
    |> BIND(FUNCTION(x): RETURN safeDivide(x, 0.0) END)    // None
    |> BIND(FUNCTION(x): RETURN safeSquareRoot(x) END)      // None (skipped)
```

### Side Effects Management

FP pushes side effects to the edges of the program, keeping the core logic pure.

```
PURE CORE WITH IMPURE SHELL

+------------------------------------------------------+
|  IMPURE SHELL (I/O, network, database, user input)   |
|                                                       |
|  +------------------------------------------------+  |
|  |  PURE CORE (business logic, transformations)   |  |
|  |                                                |  |
|  |  - All functions are pure                      |  |
|  |  - No I/O, no mutation                         |  |
|  |  - Easy to test and reason about               |  |
|  |                                                |  |
|  +------------------------------------------------+  |
|                                                       |
|  Side effects happen HERE:                            |
|  - Read user input                                    |
|  - Query databases                                    |
|  - Write to files/network                             |
|  - Display output                                     |
+------------------------------------------------------+
```

```
// Pure core
FUNCTION calculateDiscount(price: Float, tier: String) -> Float:
    MATCH tier:
        CASE "gold":     RETURN price * 0.20
        CASE "silver":   RETURN price * 0.10
        CASE "bronze":   RETURN price * 0.05
        DEFAULT:         RETURN 0.0
    END MATCH
END FUNCTION

FUNCTION applyDiscount(price: Float, discount: Float) -> Float:
    RETURN price - discount
END FUNCTION

// Impure shell
FUNCTION processOrder() -> VOID:
    SET price = READ_FROM_DATABASE("order_price")       // impure: I/O
    SET tier = READ_FROM_DATABASE("customer_tier")       // impure: I/O
    SET discount = calculateDiscount(price, tier)        // pure
    SET finalPrice = applyDiscount(price, discount)      // pure
    WRITE_TO_DATABASE("final_price", finalPrice)         // impure: I/O
    SEND_EMAIL("Your total: $" + TO_STRING(finalPrice))  // impure: I/O
END FUNCTION
```

## FP vs OOP Comparison

```
+-------------------+----------------------------+----------------------------+
| Aspect            | Object-Oriented            | Functional                 |
+-------------------+----------------------------+----------------------------+
| Core unit         | Object (state + behavior)  | Function (transformation)  |
| State management  | Mutable internal state     | Immutable data             |
| Abstraction       | Classes and interfaces     | Functions and composition  |
| Polymorphism      | Method dispatch            | Higher-order functions     |
| Code reuse        | Inheritance, composition   | Composition, currying      |
| Side effects      | Managed via encapsulation  | Pushed to program edges    |
| Concurrency       | Locks, synchronization     | Natural (no shared state)  |
| Testing           | Mock objects, DI           | Simple input/output        |
| Data flow         | Messages between objects   | Pipelines of transforms    |
+-------------------+----------------------------+----------------------------+
```

Neither paradigm is universally better. Most modern software blends both approaches:
using objects for domain modeling and functional techniques for data transformations.

## Common Pitfalls and Best Practices

### Pitfalls

1. **Over-abstraction**: Creating deeply nested function compositions that are harder to
   read than the imperative equivalent.

2. **Performance of immutability**: Creating new data structures for every change can
   be expensive without structural sharing or persistent data structures.

3. **Impure functions disguised as pure**: A function that reads from a cache, database,
   or environment variable is not pure, even if it looks like one.

4. **Excessive currying**: Currying every function makes code cryptic. Use it when partial
   application provides genuine clarity.

5. **Ignoring readability**: Chaining too many map/filter/reduce operations in a single
   pipeline can obscure the logic. Break long pipelines into named steps.

### Best Practices

1. **Start with pure functions**: Write the core logic as pure functions first, then
   add the impure shell around it.
2. **Name intermediate results**: In long pipelines, assign intermediate values to
   descriptively named variables.
3. **Use immutable data by default**: Only introduce mutation when profiling shows it
   is necessary for performance.
4. **Compose small functions**: Build complex behavior from simple, well-tested pieces.
5. **Separate concerns**: Keep data transformation (pure) separate from I/O (impure).
6. **Document function signatures**: Clear types on inputs and outputs serve as
   documentation and prevent misuse.

## Real-World Applications

- **Data processing pipelines**: ETL workflows use map/filter/reduce to transform data.
- **Concurrent systems**: Immutability eliminates race conditions in parallel code.
- **Reactive programming**: Event streams are modeled as functional transformations.
- **Configuration as code**: Pure functions generate infrastructure configurations.
- **Financial calculations**: Referential transparency ensures audit-safe computations.
- **Compiler design**: Abstract syntax trees are processed with recursive pure functions.

## Related Topics

- [Functions and Scope](functions_and_scope.md) -- Foundations: first-class functions, closures
- [Object-Oriented Programming](object_oriented_programming.md) -- Contrasting paradigm
- [Error Handling](error_handling.md) -- Option/Result types from FP for error management
- [Control Flow](control_flow.md) -- Replacing loops with functional alternatives

## Practice Problems

1. **Pure function audit**: Given a set of five functions, identify which are pure and
   which are impure, and explain why.

2. **Map/Filter/Reduce pipeline**: Using only map, filter, and reduce, compute the
   average grade of all passing students (grade >= 60) from a list of student records.

3. **Function composition**: Write a `pipe` function that composes an arbitrary number
   of functions left-to-right (opposite of mathematical composition).

4. **Curried configuration**: Write a curried `createLogger(level)(prefix)(message)`
   function and demonstrate partial application for different log levels.

5. **Immutable update**: Given a nested record (a person with an address containing a
   city), write a function that returns a new record with the city changed, without
   mutating the original.

6. **Maybe monad chain**: Write a chain of three operations that each might fail (return
   None), and show how the Maybe monad short-circuits on the first failure.

7. **Refactor to FP**: Take an imperative loop with mutation and rewrite it as a
   pipeline of pure functional transformations.
