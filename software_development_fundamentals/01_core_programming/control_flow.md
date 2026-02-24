# Control Flow

## Overview

Control flow determines the order in which statements in a program are executed. Without
control flow constructs, a program would run every instruction exactly once, from top to
bottom. Conditionals allow programs to make decisions, loops allow them to repeat work,
and control transfer statements allow them to jump to different locations in the code.

Mastering control flow is essential because virtually every meaningful program requires
branching and iteration. Whether validating user input, processing collections of data,
or implementing complex business rules, control flow constructs are the mechanisms that
give programs their logic. Understanding how these constructs compose and interact enables
developers to write clear, correct, and efficient code.

## Key Concepts

### Conditional Statements

Conditionals evaluate a boolean expression and execute different code paths based on
the result.

#### If / Else

```
IF temperature > 100 THEN
    PRINT "Water is boiling"
ELSE IF temperature > 0 THEN
    PRINT "Water is liquid"
ELSE
    PRINT "Water is frozen"
END IF
```

#### Switch / Match

Switch statements compare a single value against multiple possible matches:

```
SWITCH dayOfWeek:
    CASE "Monday":
    CASE "Tuesday":
    CASE "Wednesday":
    CASE "Thursday":
    CASE "Friday":
        PRINT "Weekday"
    CASE "Saturday":
    CASE "Sunday":
        PRINT "Weekend"
    DEFAULT:
        PRINT "Invalid day"
END SWITCH
```

#### Pattern Matching

Pattern matching extends switch statements to destructure and match complex structures:

```
MATCH shape:
    CASE Circle(radius):
        SET area = PI * radius * radius
    CASE Rectangle(width, height):
        SET area = width * height
    CASE Triangle(base, height):
        SET area = 0.5 * base * height
    DEFAULT:
        SET area = 0
END MATCH
```

### Loops

Loops repeat a block of code until a condition is met.

#### For Loop (Counted)

```
FOR i = 0 TO 9 DO
    PRINT "Iteration: " + TO_STRING(i)
END FOR
```

#### While Loop (Pre-condition)

The condition is checked before each iteration. The body may never execute.

```
SET attempts = 0
WHILE attempts < MAX_RETRIES DO
    SET success = TRY_CONNECT()
    IF success THEN
        BREAK
    END IF
    SET attempts = attempts + 1
END WHILE
```

#### Do-While Loop (Post-condition)

The body executes at least once; the condition is checked after.

```
DO
    SET input = READ_INPUT("Enter a positive number: ")
    SET number = TO_INTEGER(input)
WHILE number <= 0
```

#### For-Each Loop (Collection Iteration)

```
SET names = ["Alice", "Bob", "Carol"]
FOR EACH name IN names DO
    PRINT "Hello, " + name
END FOR
```

### Loop Control Statements

#### Break

Exits the innermost loop immediately:

```
FOR EACH item IN inventory DO
    IF item.name = searchTarget THEN
        PRINT "Found: " + item.name
        BREAK
    END IF
END FOR
```

#### Continue

Skips the rest of the current iteration and proceeds to the next:

```
FOR EACH student IN students DO
    IF student.grade = "INCOMPLETE" THEN
        CONTINUE
    END IF
    PROCESS_GRADE(student)
END FOR
```

### Guard Clauses

Guard clauses handle edge cases early, reducing nesting and improving readability.

```
// Without guard clauses (deeply nested)
FUNCTION processOrder(order: Order) -> Result:
    IF order IS NOT NULL THEN
        IF order.items.length > 0 THEN
            IF order.payment IS VALID THEN
                // actual logic here
                RETURN SUCCESS
            END IF
        END IF
    END IF
    RETURN FAILURE
END FUNCTION

// With guard clauses (flat and clear)
FUNCTION processOrder(order: Order) -> Result:
    IF order IS NULL THEN RETURN FAILURE END IF
    IF order.items.length = 0 THEN RETURN FAILURE END IF
    IF order.payment IS NOT VALID THEN RETURN FAILURE END IF

    // actual logic here
    RETURN SUCCESS
END FUNCTION
```

### Short-Circuit Evaluation

Logical operators stop evaluating as soon as the result is determined.

```
// AND short-circuit: if left is FALSE, right is never evaluated
IF user IS NOT NULL AND user.isActive THEN
    GRANT_ACCESS(user)
END IF

// OR short-circuit: if left is TRUE, right is never evaluated
SET displayName = user.nickname OR user.fullName OR "Anonymous"
```

This is critical for safety: checking for null before accessing a member prevents
null reference errors.

## Pseudocode Examples

### Nested Conditionals with Early Return

```
FUNCTION categorizeAge(age: Integer) -> String:
    IF age < 0 THEN RETURN "Invalid" END IF
    IF age < 13 THEN RETURN "Child" END IF
    IF age < 18 THEN RETURN "Teenager" END IF
    IF age < 65 THEN RETURN "Adult" END IF
    RETURN "Senior"
END FUNCTION
```

### Searching a Collection

```
FUNCTION linearSearch(items: List<Integer>, target: Integer) -> Integer:
    FOR i = 0 TO LENGTH(items) - 1 DO
        IF items[i] = target THEN
            RETURN i
        END IF
    END FOR
    RETURN -1    // not found
END FUNCTION
```

### Accumulation Pattern

```
FUNCTION sumPositive(numbers: List<Integer>) -> Integer:
    SET total = 0
    FOR EACH n IN numbers DO
        IF n > 0 THEN
            SET total = total + n
        END IF
    END FOR
    RETURN total
END FUNCTION
```

### Nested Loops

```
FUNCTION multiplicationTable(size: Integer) -> VOID:
    FOR row = 1 TO size DO
        SET line = ""
        FOR col = 1 TO size DO
            SET line = line + FORMAT(row * col, width=4)
        END FOR
        PRINT line
    END FOR
END FUNCTION
```

## Visual Representation

### If / Else Flowchart

```
            +-------------+
            |   START     |
            +------+------+
                   |
                   v
          +--------+--------+
          | Evaluate        |
          | condition       |
          +--------+--------+
                   |
           +-------+-------+
           |               |
        [TRUE]          [FALSE]
           |               |
    +------+------+  +-----+------+
    | Execute     |  | Execute    |
    | THEN block  |  | ELSE block |
    +------+------+  +-----+------+
           |               |
           +-------+-------+
                   |
                   v
            +------+------+
            |    END      |
            +-------------+
```

### While Loop Flowchart

```
            +-------------+
            |   START     |
            +------+------+
                   |
                   v
          +--------+--------+
     +--->| Evaluate        |
     |    | condition       |
     |    +--------+--------+
     |             |
     |      +------+------+
     |      |             |
     |   [TRUE]        [FALSE]
     |      |             |
     | +----+----+        |
     | | Execute |        |
     | | body    |        |
     | +----+----+        |
     |      |             |
     +------+             |
                          v
                   +------+------+
                   |    END      |
                   +-------------+
```

### Switch / Match Flowchart

```
            +---------------+
            | Evaluate      |
            | expression    |
            +-------+-------+
                    |
       +------------+------------+----------+
       |            |            |          |
   [Case A]    [Case B]    [Case C]   [Default]
       |            |            |          |
  +----+---+  +----+---+  +----+---+  +---+----+
  | Block  |  | Block  |  | Block  |  | Block  |
  | A      |  | B      |  | C      |  | Dflt   |
  +----+---+  +----+---+  +----+---+  +---+----+
       |            |            |          |
       +------------+------------+----------+
                    |
                    v
               +----+----+
               |  END    |
               +---------+
```

## Common Pitfalls and Best Practices

### Pitfalls

1. **Infinite loops**: Forgetting to update the loop variable or exit condition causes
   the program to hang. Always ensure progress toward termination.

2. **Off-by-one errors**: Using `<=` instead of `<` (or vice versa) in loop bounds.
   Clarify whether ranges are inclusive or exclusive.

3. **Deep nesting**: More than 3 levels of nested conditionals become hard to read.
   Use guard clauses, early returns, or extract helper functions.

4. **Fall-through in switch**: Some environments continue into the next case unless
   an explicit break or return is present. Be deliberate about fall-through behavior.

5. **Mutating collections during iteration**: Adding or removing elements from a
   collection while iterating over it leads to unpredictable behavior.

### Best Practices

1. **Prefer for-each over indexed loops** when you do not need the index.
2. **Use guard clauses** to handle error cases early and keep the main logic flat.
3. **Limit loop body size**: If a loop body exceeds 10-15 lines, extract a function.
4. **Name boolean conditions**: `SET isEligible = age >= 18 AND hasConsent` is clearer
   than embedding the expression directly in an `IF` statement.
5. **Use pattern matching** when available instead of long if-else chains.

## Real-World Applications

- **Input validation**: Guard clauses reject bad data before processing.
- **Menu systems**: Switch statements route user choices to handlers.
- **Data processing pipelines**: For-each loops transform collections of records.
- **Retry logic**: While loops with counters implement retry-with-backoff patterns.
- **State machines**: Switch/match on state values drives event-driven systems.
- **Game loops**: While loops drive the main update-render cycle in interactive software.

## Related Topics

- [Variables and Data Types](variables_and_data_types.md) -- Boolean expressions in conditions
- [Functions and Scope](functions_and_scope.md) -- Extracting complex control flow into functions
- [Error Handling](error_handling.md) -- Control flow for exceptional conditions
- [Functional Programming](functional_programming.md) -- Replacing loops with map/filter/reduce

## Practice Problems

1. **FizzBuzz**: Write pseudocode that prints numbers 1 through 100 but replaces multiples
   of 3 with "Fizz", multiples of 5 with "Buzz", and multiples of both with "FizzBuzz".

2. **Number guessing game**: Write a do-while loop that asks the user to guess a secret
   number, providing "too high" or "too low" hints until they guess correctly.

3. **Prime checker**: Write a function using a for loop and early return to determine
   whether a given integer is prime.

4. **Flatten nested conditions**: Take a block of code with 4+ levels of nesting and
   refactor it using guard clauses.

5. **Pattern match shapes**: Use pattern matching to compute both the area and perimeter
   of Circle, Rectangle, and Triangle shapes.

6. **Break vs flag**: Rewrite a loop that uses a boolean flag variable to instead use a
   break statement, and discuss which approach is clearer.
